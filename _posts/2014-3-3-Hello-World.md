---
layout: post
title: TypeScript Ability Scores for Pathfinder 2e (or other D&D like TTRPGS...)
published: true
---
## The Ability Score Breakdown

An "Ability Score" is just a combination of two things: text (known as a string) describing the score and the actual numerical value. In [TypeScript](https://www.typescriptlang.org/) we can type an AbilityScore using...

	interface AbilityScore {
    	ability: string;
    	score: number;
	}	

or using...

	type AbilityScore = {
    	ability: string;
    	score: number;
	}
   
If you're curious how those two are different check out the official docs on [Interfaces vs. Type Aliases](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces). I'll be using them interchangably and only call it out again if it makes a radical difference.

Now create an AbilityScore in TypeScript:

	const abilityScore: AbilityScore = { ability: "Strength", score: 18 };
    console.log(abilityScore) // Show the ability score in the logs!

We've now created and defined an AbilityScore, but you never have just one!

## Typing Ability Scores, plural

We need to define all six of our stats and create some interfaces/typing around it so let's create another type.

	type AbilityScoreArray = {
      Strength: AbilityScore;
      Dexterity: AbilityScore;
      Constitution: AbilityScore;
      Intelligence: AbilityScore;
      Wisdom: AbilityScore;
      Charisma: AbilityScore;
      [key: string]: AbilityScore;
	};
    
 This is very similar to what we've done above, but that last line, `[key: string]: AbilityScore`, is new. This is called an [index signature](https://www.typescriptlang.org/docs/handbook/2/objects.html#index-signatures) and it allows us to reference the index of the `AbilityScoreArray` object using the name of the property. Let's see an example.
 
 	const abilityScoreArray: AbilityScoreArray = {
    	Strength: abilityScore, // this assumes our varibale from above is still in scope!
        Dexterity: { score: 16, ability: "Dexterity" },
        Constitution: { score: 20, ability: "Constitution" },
        Intelligence: { score: 6, ability: "Intelligence" },
        Wisdom: { score: 18, ability: "Wisdom" },
        Charisma: { score: 8, ability: "Charisma" }
    };
    console.log(abilityScoreArray.Strength.ability); // "Strength"
	console.log(abilityScoreArray.Strength.score); // 18
    console.log(abilityScoreArray["Charisma"]); // { score: 8, ability: "Charisma" }
    
We now have a pretty cool type we can use for our Ability Scores. But we're playing Pathfinder2e (if not, you should be ;) ) and there are only ever those six scores allowed. However, as currently written, the following is valid TypeScript code:

	const badAbilityScoreArray: AbilityScoreArray = {
    	Strength: abilityScore, // this assumes our varibale from above is still in scope!
        Dexterity: { score: 16, ability: "NOT DEX!" },
        Constitution: { score: 20, ability: "Constitution" },
        Intelligence: { score: 6, ability: "Intelligence" },
        Wisdom: { score: 18, ability: "Wisdom" },
        Charisma: { score: 8, ability: "Charisma" }
    };
    
Did you spot the problem?

`Dexterity: { score: 16, ability: "NOT DEX!" },` is prefectly acceptable TypeScript code because `"NOT DEX!"` is a `string`. So how do we make sure that this can't happen? We can futher type our AbilityScore interface to only allow certain strings. 

	enum Ability {
      Strength = "Strength",
      Dexterity = "Dexterity",
      Constitution = "Constitution",
      Intelligence = "Intelligence",
      Wisdom = "Wisdom",
      Charisma = "Charisma",
	}

An enum is a pre-defined set of values, which I think works perfectly for what we need, because now we can do this:

	interface AbilityScoreStrict {
      ability: Ability;
      score: number;
	}	

Which means that the `ability` property of our `AbilityScoreStrict` **has** to be one of the strings we defined in our `Ability` enum. Now this doens't prevent someone from creating somthing where the wrong Ability is assigned to the wrong ability score, but we can atleast save ourselves from some typos using tab-completion in our code editor.

![Showing code completion in an editor using the Ability enum.]({{site.baseurl}}/_posts/Screen Shot 2021-05-13 at 9.22.51 PM.png)

## Ability Score modifiers

Having this awesome type for our Ability Scores is great, but all of our actions/stats are using the Modifier in the math.

### The Equation

According to [this rpg stackexchange question](https://rpg.stackexchange.com/questions/71076/is-there-an-ability-modifier-equation), the formula for calculating an Ability Score modifier from the score itself is `(Ability Score - 10) / 2` rounded down. This leads us to [this table commonly seen in RPG rule books,](https://2e.aonprd.com/Rules.aspx?ID=77) where the modifier increases every even number. 

Now that we have the equation, let's move on to...

### Writing a TypeScript Function to calculate an Ability Score modifier

Here's our whole function, we'll dissect it as we go.
	
    function CalculateModifierFromAbilityScore(abilityScore: AbilityScore): number {
      const scoreMinus10: number = abilityScore.score - 10;
      const dividedBy2: number = scoreMinus10 / 2;
      const roundedDown = Math.floor(dividedBy2);
      return roundedDown;
	}

With `function CalculateModifierFromAbilityScore(abilityScore: AbilityScore): number {` we name our function and indicate that we are passing in a parameter of type `AbilityScore` and will be returning a `number` value.

The next two lines should explain themselves with the variable naming, but `const roundedDown = Math.floor(dividedBy2);` could use some clarification. [`Math.floor`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/floor) "_Returns the greatest integer less than or equal to its numeric argument._" or simply put, rounds down.

Lastly, our function returns the modifier as a `number` value.
Our code can now do something like...

	const constitutionModifier: number = CalculateModifierFromAbilityScore(abilityScoreArray["Constitution"]);
	console.log(constitutionModifier); // 5
    
## Thank you!

For stopping by and reading my humble blog. I hope you find value in this. Please check out my [TypeScript React Native project of the same name as the site](https://github.com/mirkoRainer/RulesLawyer) if this sort of thing interests you. I'm always open to PRs, collaboration, and/or critique. Or reach out to me at mirkoRainer@outlook.com if you have any questions or comments.

_**May your dice always land on 20!**_

