---

name: validating-node-component
description: Validate either node or component or both in Cocos Creator 3.8.X
compatibility: opencode
---

# Main Utility Functions for validating node and component

**NOTE:**

- DO NOT use the onEnable/onDisable flow to re/unregister event listeners, use onDestroy and this utility functions instead
- Always look for these functions or similar utility functions in the codebase, don't reinvent the wheel
- If you don't find them, ask user for permission to create them
- Because this checks for both `isValid` and `!!target`, it should be used only for operations that required both `!!target` and `target.isValid`

**Implementation:**

```ts

export function isComponentAndNodeValid(targetComponent: Component) {
  return targetComponent && targetComponent.isValid && isNodeValid(targetComponent.node);
}

export function isNodeValid(targetNode: Node) {
  return targetNode && targetNode.isValid;
}

```

**Usage:**

```ts

// ✅ EXCELLENT: Validate component and node before unregistering event listener
if (isComponentAndNodeValid(this.targetComponent)) {
  this.targetComponent.node.off(CUSTOM_EVENT, this.calllbackEvent, this);
}

if (isNodeValid(this.targetNode)) {
  this.targetNode.off(CUSTOM_EVENT, this.calllbackEvent, this);
}

// ❌ WRONG: Use as null/undefined check, because it also checks for `isValid` so should reserve only for operations that required both `!!target` and `target.isValid`
if (isNodeValid(this.targetNode)) {
    this.targetNode.setPosition(0, 0, 0);
}

```

```ts

// ✅ EXCELLENT: Validate component and node before unregistering event listener within onDestroy
protected onDestroy(): void {
  if (isComponentAndNodeValid(this.targetComponent)) {
    this.targetComponent.node.off(CUSTOM_EVENT, this.calllbackEvent, this);
  }

  if (isNodeValid(this.targetNode)) {
    this.targetNode.off(CUSTOM_EVENT, this.calllbackEvent, this);
  }
}

// ❌ WRONG: Using the onEnable/onDisable to validate component and node before enabling/disabling the component
protected onEnable(): void {
  if (isComponentAndNodeValid(this.targetComponent)) {
    this.targetComponent.node.on(CUSTOM_EVENT, this.calllbackEvent, this);
  }

  if (isNodeValid(this.targetNode)) {
    this.targetNode.on(CUSTOM_EVENT, this.calllbackEvent, this);
  }
}

// ❌ WRONG: Using the onEnable/onDisable to validate component and node before enabling/disabling the component
protected onEnable(): void {
  this.targetComponent.node.on(CUSTOM_EVENT, this.calllbackEvent, this);
}

protected onDisable(): void {
  if (isComponentAndNodeValid(this.targetComponent)) {
    this.targetComponent.node.off(CUSTOM_EVENT, this.calllbackEvent, this);
  }
}
```
