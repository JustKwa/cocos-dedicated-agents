---
name: cocos-create-component
description: Create a new component in Cocos Creator 3.8.X using Entity-Component (EC) System
compatibility: opencode
---

# Cocos Creator Component System

Cocos Creator uses an Entity-Component (EC) architecture where:

- **Node** = Entity (game object container)
- **Component** = Behavior/functionality attached to Node
- **Scene** = Collection of Node hierarchies

```typescript
import { _decorator, Component, Node } from 'cc';
const { ccclass, property } = _decorator;

// ✅ EXCELLENT: Complete component structure
@ccclass('PlayerController')
export class PlayerController extends Component {
    // @property decorator exposes fields to Inspector
    // NOTE: inspector property should be name `camelCase`
    @property(Node)
    private targetNode: Node | null = null;

    // NOTE: Private fields should be name `m` prefix
    private mCurrentHealth: number = 100;
    // NOTE: Static readonly fields name should be in uppercase
    private static readonly MAX_HEALTH: number = 100;
}
```

## @ccclass Decorator

```typescript
import { _decorator, Component } from 'cc';
const { ccclass } = _decorator;

// ✅ EXCELLENT: @ccclass with explicit name
@ccclass('GameManager')
export class GameManager extends Component {
    // Component implementation
}

// ❌ WRONG: Missing @ccclass decorator
export class GameManager extends Component {
    // Won't work - Cocos can't serialize this component
}

// ❌ WRONG: Not extending Component
@ccclass('GameManager')
export class GameManager {
    // Won't work - must extend Component
}
```

## @property Decorator

```typescript
import { _decorator, Component, Node, Sprite, Label, CCFloat, CCString, CCBoolean } from 'cc';
const { ccclass, property } = _decorator;

@ccclass('PropertyExamples')
export class PropertyExamples extends Component {
    // ✅ EXCELLENT: Node reference
    @property(Node)
    private playerNode: Node;

    // ✅ EXCELLENT: Component reference
    @property(Sprite)
    private spriteComponent: Sprite;

    // ✅ EXCELLENT: Use explicit `CCFloat` type for float properties
    @property(CCFloat)
    private moveSpeed: number = 100.0;

    // ✅ EXCELLENT: Should import appropriate type from `cc` module
    @property(CCString)
    private playerName: string = 'Player';

    @property(CCBoolean)
    private enableDebug: boolean = false;

    // ✅ EXCELLENT: Array of inspector properties of any given type
    @property([Node])
    private enemyNodes: Node[] = [];

    // ✅ EXCELLENT: Enum property
    @property({ type: Enum(GameState) })
    private currentState: GameState = GameState.LOADING;

    // ✅ EXCELLENT: Property with custom display name and tooltip
    @property({
        type: Number,
        displayName: 'Movement Speed',
        tooltip: 'Player movement speed in units per second',
        min: 0,
        max: 500,
        step: 10,
    })
    private speed: number = 100;
}

// ❌ WRONG: Property without type
@property
private playerNode: Node; // Won't serialize correctly
```

## Component Lifecycle Methods

### 1. onLoad()/onDestroy() - Initialization/Cleanup

```typescript
import { _decorator, Component, Node, assert, UITransform } from 'cc';
const { ccclass, property } = _decorator;

@ccclass('GameManager')
export class GameManager extends Component {
    @property(Node)
    private playerNode: Node;

    private mUITransform: UITransform;

    // ✅ EXCELLENT: onLoad for initialization and validation
    protected onLoad(): void {
        // Validate required references
        assert(this.playerNode, '[GameManager]: playerNode is null or not set');

        // Cache references
        this.mUITransform = this.node.getComponent(UITransform);

        // Listen to events, use arrow function
        this.mUITransform.on(UITransform.EventType.SIZE_CHANGED, this.onSizeChanged, this);
    }

    private onSizeChanged = () => { }
}

// ❌ WRONG: Heavy operations in onLoad
protected onLoad(): void {
    // Avoid expensive operations - onLoad should be fast
    this.loadAllLevelData(); // Should be async in start()
    this.generateProceduralContent(); // Too expensive for onLoad
}
```

### 2. start() - Post-Initialization

```typescript
import { _decorator, Component, Node } from 'cc';
const { ccclass, property } = _decorator;

@ccclass('PlayerController')
export class PlayerController extends Component {
    @property(Node)
    private enemyManagerNode: Node;

    private enemyManager: EnemyManager;

    protected onLoad(): void {
        assert(this.enemyManagerNode, '[PlayerController]: enemyManagerNode is null or not set');
        this.enemyManager = this.enemyManagerNode.getComponent(EnemyManager);
    }

    // ✅ EXCELLENT: start() for referencing other components
    protected start(): void {
        this.setupPlayerBasedOnEnemies();
        // Use void operator to explicitly indicate that async function that's not awaited is intentional
        void this.loadPlayerDataAsync();
    }

    private setupPlayerBasedOnEnemies(): void {
        const enemyCount = this.enemyManager.getEnemyCount();
        this.adjustDifficultyBasedOnEnemies(enemyCount);
    }

    private async loadPlayerDataAsync(): Promise<void> { }
}
```

### 4. update(dt) - Per-Frame Logic

```typescript
import { _decorator, Component, Node, Vec3 } from 'cc';
const { ccclass, property } = _decorator;

@ccclass('PlayerMovement')
export class PlayerMovement extends Component {
    @property(Number)
    private readonly moveSpeed: number = 100;

    private readonly tempVec3: Vec3 = new Vec3();
    private inputDirection: Vec3 = new Vec3(1, 0, 0);

    // ✅ EXCELLENT: Efficient update implementation
    protected update(dt: number): void {
        // Reuse preallocated vector
        this.node.getPosition(this.tempVec3);

        // Calculate movement
        this.tempVec3.x += this.inputDirection.x * this.moveSpeed * dt;
        this.tempVec3.y += this.inputDirection.y * this.moveSpeed * dt;

        // Apply new position
        this.node.setPosition(this.tempVec3);
    }
}

// Throttled expensive operations
@ccclass('AIController')
export class AIController extends Component {
    private mFrameCount: number = 0;
    private static readonly AI_UPDATE_INTERVAL: number = 10;

    // ✅ EXCELLENT: Throttle expensive operations
    protected update(dt: number): void {
        this.frameCount++;

        // Cheap operations every frame
        this.moveTowardsTarget(dt);

        // Expensive AI decisions every 10 frames
        if (this.frameCount % AIController.AI_UPDATE_INTERVAL === 0) {
            this.updateAIDecision();
        }
    }

    private moveTowardsTarget(dt: number): void {
        // Simple movement calculation
    }

    private updateAIDecision(): void {
        // Complex AI logic
    }
}

// ❌ WRONG: Allocations in update
protected update(dt: number): void {
    const currentPos = this.node.position.clone(); // Allocates every frame!
    currentPos.x += this.moveSpeed * dt;
    this.node.setPosition(currentPos);
}

// ❌ WRONG: Expensive operations every frame
protected update(dt: number): void {
    this.recalculatePathfinding(); // A* algorithm 60 times per second!
    this.updateComplexAI(); // Too expensive for every frame
}

// ❌ WRONG: Component lookups in update
protected update(dt: number): void {
    const sprite = this.node.getComponent(Sprite); // Cache this in onLoad!
    sprite?.doSomething();
}
```

### 5. lateUpdate(dt) - Post-Update Logic

```typescript
import { _decorator, Component, Node, Camera } from 'cc';
const { ccclass, property } = _decorator;

@ccclass('CameraFollow')
export class CameraFollow extends Component {
    @property(Node)
    private target: Node;

    @property(Camera)
    private camera: Camera;

    // ✅ EXCELLENT: lateUpdate for camera following
    // Runs after all update() calls, ensuring target has moved
    protected lateUpdate(dt: number): void {
        if (!this.target || !this.camera) return;

        // Follow target position after target has been updated
        const targetPos = this.target.position;
        this.camera.node.setPosition(targetPos.x, targetPos.y, this.camera.node.position.z);
    }
}

// ✅ GOOD: lateUpdate for UI that depends on game state
@ccclass('HealthBarUpdater')
export class HealthBarUpdater extends Component {
    @property(Node)
    private healthBar: Node;

    private mPlayerHealth: number = 100;

    // Health is updated in PlayerController.update()
    // UI is updated in lateUpdate() to reflect final health value
    protected lateUpdate(dt: number): void {
        if (!this.healthBar) return;

        const healthPercentage = this.mPlayerHealth / 100;
        this.healthBar.scale = new Vec3(healthPercentage, 1, 1);
    }
}

// ❌ WRONG: Using lateUpdate for regular logic
protected lateUpdate(dt: number): void {
    // This should be in update(), not lateUpdate()
    this.movePlayer(dt);
}
```

### 6. onDestroy() - Cleanup

```typescript
import { _decorator, Component, Node, UITransform } from 'cc';
const { ccclass, property } = _decorator;

function isComponentAndNodeValid(component: Component): boolean {
    return component !== null && component.isValid && isNodeValid(component.node);
}

function isNodeValid(node: Node): boolean {
    return node !== null && node.isValid;
}

@ccclass('ResourceManager')
export class ResourceManager extends Component {
    private mUITransform: UITransform;

    private readonly loadedAssets: Map<string, Asset> = new Map();
    private readonly eventListeners: Set<Function> = new Set();
    private readonly scheduledCallbacks: Set<Function> = new Set();

    // ✅ EXCELLENT: Complete cleanup in onDestroy
    protected onDestroy(): void {
        // Lookup and use functions (isComponentAndNodeValid or isNodeValid) that checks if component and node are valid, if doesn't exist ask user for permission to create said utility 
        if (isComponentAndNodeValid(this.mUITransform)) {
            this.mUITransform.off(UITransform.EventType.SIZE_CHANGED, this.onSizeChanged, this);
        }

        this.node.off(Node.EventType.TOUCH_START);

        // Clear collections
        this.eventListeners.clear();
        this.scheduledCallbacks.clear();

        // Release loaded assets
        for (const [id, asset] of this.loadedAssets) {
            asset.decRef();
        }
        this.loadedAssets.clear();

        // Unschedule all callbacks
        this.unscheduleAllCallbacks();

        // Clear any references to prevent memory leaks
        this.clearReferences();
    }

    private clearReferences(): void { }
}

// ❌ WRONG: Missing cleanup
protected onDestroy(): void {
    // Forgot to unregister events - memory leak!
    // Forgot to release assets - memory leak!
    // Forgot to unschedule callbacks - may cause errors!
}

// ❌ WRONG: Incomplete cleanup
protected onDestroy(): void {
    this.loadedAssets.clear(); // Cleared map but didn't decRef assets!
}
```

## Component Execution Order

```typescript
// Execution order when scene loads:
// 1. All components: onLoad() (in hierarchy order)
// 2. All components: start() (in hierarchy order)
// 3. All components: onEnable() (if not already enabled)
// 4. Begin frame loop:
//    - All components: update(dt)
//    - All components: lateUpdate(dt)
// 5. When component disabled:
//    - Component: onDisable()
// 6. When component destroyed:
//    - Component: onDestroy()

// ✅ EXCELLENT: Method blocks organization
@ccclass('CompleteLifecycle')
export class CompleteLifecycle extends Component {
// ✅ EXCELLENT: Use #region and #endregion to organize method blocks
    // #region Public Fields

    // #endregion

    // #region Inspector Properties

    //✅ EXCELLENT: Keep spacing between #region and #endregion
    @property(Node)
    private playerNode: Node;

    // #endregion

    // #region Private Fields

    // #endregion

    // #region Component Lifecycle Methods

    // #endregion

    // #region Public Methods

    // #endregion

    // #region Private Methods

    // #endregion
}
```

// ✅ EXCELLENT: Create sub regions depending on functions
// ❌ WRONG: Don't create sub regions if it's only one function for that group

```ts
    // #region Public Methods

    // #region Static Public Methods

    public static getInstance() { } 

    public static calculateSomething(a: number, b: number) { }

    // #endregion

    // #endregion
```

## Summary: Component System Checklist

**Lifecycle Implementation:**

- [ ] onLoad() - Validate required references, initialize state, cache references
- [ ] start() - start async operations, start any operations that need other scripts to be loaded
- [ ] update(dt) - Per-frame logic (zero allocations)
- [ ] lateUpdate(dt) - Post-update logic (camera, UI)
- [ ] onDestroy() - Release resources, clear references, unschedule callbacks, unregister listeners

**Best Practices:**

- [ ] Validate required @property references in onLoad() using assert or throw exceptions
- [ ] Cache component references (don't lookup in update)
- [ ] Zero allocations in update/lateUpdate
- [ ] Always unregister listeners in onDestroy
