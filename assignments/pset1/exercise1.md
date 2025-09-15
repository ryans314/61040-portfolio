# Exercise 1: Reading a Concept

## 1. Invariants

**What are two invariants of the state?**

1. Item counts - the count on a request's item will never be less than 0. 

2. Requests and Purchases - each purchase corresponds to exactly 1 request

**Which one is more important and why?**

The Request and Purchase invariant is more important, since it directly relates to the operational principle (If a purchase could be made without relating to a request, then there is no connection to the registry). 

**Which action's design is most affected by the invariant, and how does it preserve the invariant?**

The invariant most directly affects the design of the purchase action, which preserves the invariant by taking as input a registry and an item, and requiring that a request exists in that registry for that count.

## 2. Fixing an action

**Identify an action that potentially breaks the invariant and explain how that may happen?**

The removeItem action may violate the above invariant by removing a request that has already has a purchase associated with it. This would result in a purchase existing without an associated request. 

This could be fixed by adding a requirement to removeItem that the item has no purchases made for it yet.

## 3. Inferring Behavior

**Can a registry be repeatedly opened and closed? Why would this be allowed?**

Yes, a registry can be closed and opened repeatedly, since each action only toggles the active flag. 

A reason to allow this would be in case the registry owners realize after closing the registry that there are additional things that they want/need related to the occasion. Another reason for additional toggling would be for repeat occassions, like baby showers (for multiple pregnancies).

## 4. Registry deletion

**Does it matter (in practice) that there is no action to delete a registry?**

No, since there is a way to mark a registry active/inactive, it can be treated as deleted when implemented - meaning that no actions can be made with it, and it can potentially be made no longer visible when inactive.

## 5. Queries

**What are two common queries likely to be executed against the concept state?**

The most frequent queries to be made are likely addItem and purchase. Every item on the registry will need to be added by the owners, and every gift will need to be purchased by the gift givers.

## 6. Hiding Purchases

**How could you augment the concept spec to allow the recipient to choose not to see purchases?**

- Add a purchase_visible Flag to each Registry
- Change create action to set purchase_visible to True
- Add a toggleVisibility(registry: Registry) action, which sets purchase_visible to not purchase_visible

With the understanding that if the purchase_visible flag is False, then the purchases will be hidden from the recipient.

## 7. Generic Types

**Explain why it would be preferable for the Item type to be populated by SKU codes rather than by name/description/prices**

SKU codes are unlikely to change arbitrarily, whereas item prices, descriptions, and even names can change or be easily misrepresented. Additionally, as long as the SKU codes point to an item name+description+price, it is more effective to link an item to a single SKU than to multiple separate item features. 


