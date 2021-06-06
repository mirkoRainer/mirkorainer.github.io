---
published: false
---
## Pathfinder 2e Checks and DCs Using TypeScript

Before we write any code, let's define our requirements for what a Check and DC is, and how we can calculate/resolve a Check. Then we'll start putting some TypeScript onto the page.

From the [Core Rulebook pg. 443](https://2e.aonprd.com/Rules.aspx?ID=314):

> _When success isn’t certain—whether you’re swinging a sword at a foul beast, attempting to leap across a chasm, or straining to remember the name of the earl’s second cousin at a soiree—you’ll attempt a check. Pathfinder has many types of checks, from skill checks to attack rolls to saving throws, but they all follow these basic steps._

**1. Roll a d20 and identify the modifiers, bonuses, and penalties that apply.**
**1. Calculate the result.**
**1. Compare the result to the difficulty class (DC).**
**1. Determine the degree of success and the effect.**

> _Checks and difficulty classes (DC) both come in many forms. When you swing your sword at that foul beast, you’ll make an attack roll against its Armor Class, which is the DC to hit another creature. If you are leaping across that chasm, you’ll attempt an Athletics skill check with a DC based on the distance you are trying to jump. When calling to mind the name of the earl’s second cousin, you attempt a check to Recall Knowledge. You might use either the Society skill or a Lore skill you have that’s relevant to the task, and the DC depends on how common the knowledge of the cousin’s name might be, or how many drinks your character had when they were introduced to the cousin the night before._

> _No matter the details, for any check you must roll the d20 and achieve a result equal to or greater than the DC to succeed. Each of these steps is explained below. We loved with a love that was more than love_

We'll focus mainly on the four steps outlined in bold as a template to writing our methods/functions, but first let's identify the types we need for our methods.

### Creating Types

We need a way to enumerate the outcomes. There are only four of these so this is relatively straightforward with a TypeScript enum.
```ts
enum CheckOutcome {
    CriticalSuccess = "Critical Success",
    Success = "Success",
    Failure = "Failure",
    CriticalFailure = "Critical Failure",
}
```
We need a type to help us track Bonuses and Penalties of the various types. We use Bonus for these two interfaces, but we'll be using it for Penalties too since a Penalty is just a Bonus with a negative value.
```ts
enum BonusType {
    Proficiency = "Proficiency",
    Circumstance = "Circumstance",
    Status = "Status",
    Item = "Item",
    Armor = "Armor", // For use in Armor Check Penalties and Speed Penalties.
    Untyped = "Untyped",
}
interface Bonus {
    type: BonusType;
    appliesTo: string; // A key to determine if the Bonus applies to the check.
    amount: number;
    source: string;
}
```
We now have Bonuses and Penalties covered, and since Modifiers don't require any special logic, we just be using the `number` primitive type for them.

