---
author: Matej Samler
title: Example of using Power FX SDK, talking to Dataverse and implementing custom functions
date: '2023-09-01T15:00:00+0200'
categories:
  - Power Platform
  - Dataverse
  - SDK
  - C#
  - Power Fx
tags:
  - Power Platform
  - Dataverse
  - SDK
  - C#
  - Power Fx
---

The built in permission system in Dataverse is fine for most usecases and offers flexible way of managing access to diferent entities. However, it's not really usable for more complex authorization rules, that might be different for specific users. We also want these rules to work with runtime variables or data in the system itself, which may change frequently.

To get this flexibility, we opted for Power FX which the customizers are already familiar with and it is slowly getting used on more places in Power Platform (not just Canvas Power Apps)

Few weeks ago Low-code Dataverse plugins were introduced as a preview feature. This feature could have been used but we would have gotten limited control over impersonation / privilege elevation which is necessary in our case. Also we wanted to provide administrators with an easy-to-use UI for configuration and make the expressions to contain the least possible amount of code. This approach also doesn't allow us to use our own custom functions, which turned out to be neccessary, as you will see bellow.

Lastly, since we wanted these rules to work outside of Dataverse plugins, in our external API connected to dataverse, as well, we settled on implementing it ourselves.

Thanks to the [Power Fx GitHub repo](https://github.com/microsoft/Power-Fx), this task wasn’t as daunting as it might have looked. I would like to praise the team behind it for providing great range of tests, from which I could easily see, how to use the library.

## Problem 1: Run-time (vs. design-time) objects

Because we want to use this as a permission model, that would decide if the user were or were not allowed to perform an action, we need to have some form of context of the request. We want to be able to check, what the user wants to change, so we need to parse and use the Request object.

To do this, we add a new variable in the symbol table called RequestBody, which will be a representation of the request object the user sends. For this, we wrote a helper function that allows us to cast a Newtonsoft.Json JObject into a RecordValue, so that the expression can use it.

We can use the handy Marshaller that Power Fx repo provides, that allows us to cast primitive C# types into Power Fx FormulaTypes and FormulaValues

```csharp
if (PrimitiveValueConversions.TryGetFormulaType(objetToCast.GetType(), out FormulaType powerFxType))
    {
        return PrimitiveValueConversions.Marshal(objetToCast, powerFxType);
    }
```

We can also check if the property value inside is a JArray and cast it approprietly. The equivalent of an Array in Power Fx is a SingleColumnTable, so you can cast your array of primitives into that.

```csharp
 FormulaValue.NewSingleColumnTable(arrayOfPrimitives.Select(x => FormulaValue.New((string)x!)))
```

You might think there may have been an “obvious” choice on how to achieve the same result using the built in function “ParseJSON”, but that returns an Untyped object which you would then need to cast with functions such as Boolean or Value.

In case of update or delete requests, we can also provide a reference to the record that’s being affected with a handy object Record.

Having an object with marshalled types allows the evaluator to know exactly what types each property is, which gives us accurate check result when working with incomparable types. We shouldn’t be allowed to add a number to a string for example. Casting a JArray into a single column table also gives us the use of the in operator, which will surely come in handy.

An issue with working with objects, that are unknown at runtime is the fact that you cannot reference properties, that might not be there.

Consider the following example: We want to restrict the user from disabling a record using the api. Every other change is permitted. We could write a Power Fx expression like so:

```csharp
If(RequestBody.statuscode <> 1, true, false) //If for emphasis
```

This will work correctly if the user actually tries to disable the record, but wont work in other cases, because the statuscode will not be present in the requestbody and we’ll get an error.

```error
Errors: Error 1-3: Name isn't valid. 'statuscode' isn't recognized.
```

We can solve this problem using custom functions.

Custom functions allow us to register a C# function that can then be used in Power Fx expressions just like any other. To add a function, we need to define it's name, return type and the types of it's parameters. Here, we are creating a "ContainsKey" function, that will return a boolean and tell you, if a Record contains an attribute with a certain key.

```csharp
public class ContainsKeyFunction : ReflectionFunction
{
    public ContainsKeyFunction() : base("ContainsKey", FormulaType.Boolean, RecordType.Empty(), FormulaType.String)
    {
    }

    public static BooleanValue Execute(RecordValue collection, StringValue key)
    {
        return FormulaValue.New(collection.GetField(key.Value) is not BlankValue);
    }
}
```

Sadly, this function will not be enough, since the check doesn't like invalid names even in branches it will not reach. Writing the following expression:

```csharp
If(ContainsKey(RequestBody, "statuscode"), RequestBody.statuscode <> 1, true)
```

will give us the same error. 'statuscode' is not recognized, even though we made sure it exists before checking it. This issue is also solvable by yet another custom function.

```csharp
public class TryEqualsDecimal : ReflectionFunction
{
    public TryEqualsDecimal() : base("TryEquals", FormulaType.Boolean, RecordType.Empty(), FormulaType.String, FormulaType.Decimal)
    {
    }
    public static BooleanValue Execute(RecordValue collection, StringValue key, DecimalValue value)
    {

        if (collection.GetField(key.Value) is BlankValue) return FormulaValue.New(false);

        return FormulaValue.New(((DecimalValue)collection.GetField(key.Value)).Value == value.Value);

    }
}

public class TryEqualsString : ReflectionFunction
{
    public TryEqualsString() : base("TryEquals", FormulaType.Boolean, RecordType.Empty(), FormulaType.String, FormulaType.String)
    {
    }
    public static BooleanValue Execute(RecordValue collection, StringValue key, FormulaValue value)
    {

        if (collection.GetField(key.Value) is BlankValue) return FormulaValue.New(false);

        return FormulaValue.New(collection.GetField(key.Value) == value);

    }
}
```

Lucky for us, Power Fx custom functions support overloading by defining the same name of a function, in our case "TryEquals" and different parameters (One takes a decimal, the other a string). With this we can safely work with objects that are not known to us on design time.

```csharp
    TryEquals(RequestBody, "statuscode", 2)
```

## Problem 2: Connection with Dataverse

But let's talk about the main deal: How do we connect to Dataverse and replicate the ability of Check and Autocomplete with Power Fx? We want something similar to how Canvas Apps function.

In Cavas Apps, if you want to use a Dataverse table inside your expression, you have to add it explicitly as a data source. Once you do that, you get access to the same type checking and intelisence that you would expect. In our case, there is no way to "add a datasource". So, can we do better? Yeah.

We can use the built in tokenizer to help us visualize our expression. We can split our expression into tokens and detect the datasources that we'll require. Here's an example of an expression we can use:

```csharp
First(Accounts)
```

This will split the tokens as such:  
![tokenSplit](/uploads/2023/08/using-powerfx-with-dataverse2.png)

We can only focus on those tokens, that are identifying something (Ident).

```csharp
_powerFxEngine.Tokenize(expression).Where(x => x.Kind == TokKind.Ident);
```

After that, let's get rid of those tokens that we know we don't need. One such example are Functions. We know that the Ident "First" is not going to be a Dataverse table but a Function name. So let's get rid of known function names

```csharp
// Filter out the tokens that are present in _powerFxEngine.SupportedFunctions
var supportedTokens = tokens.Select(x => (IdentToken)x).Where(token => !_powerFxEngine.GetAllFunctionNames().Contains(token.Name));
```

Now we should only have potential table names. We can download the list of Display and Logical names of all the tables in our Dataverse and cache it for future use (using the RetrieveAllEntitiesRequest). If you specify the "EntityFilters.Entity" filter, you should only get the basic info about the table, not the whole metadata.

We then iterate through IdentTokens that we have left and if we recognize a DisplayCollectionName, we can download (and cache) the full metadata of the specific table. In short, we will parse the expression and add the "DataSources" dynamically.

Now, we need to define Marshalling for Dataverse fields. We need to map AttributeTypeCode enum to a FormulaType of Power Fx and add it into the symbols.

In our case, we created a RecordType.Empty(), to which we have added a TableType for each required Dataverse Table.

For each attribute that we have in the metadata of the required table, we cast it from it's Attribute metadata into the relevant FormulaType.

```csharp
case AttributeTypeCode.String:
    metadataType = metadataType.Add(new NamedFormulaType(logicalName, FormulaType.String, displayName));
    break;
```

In short: Create a RecordType. For each Dataverse table you are using, add a TableType into it, filled with NamedFormulaTypes for each Dataverse column. Return the RecordType and use it whereever you may need.

```csharp
// Intelisence
Intellisense.IIntellisenseResult intelisence = _powerFxEngine.Suggest(expression, GetContextFromMetadata(expression), expression.Length);

// Checking for errors
CheckResult check = _powerFxEngine.Check(expression, GetContextFromMetadata(expression));
```

Some pro tips:

- If you have fields on your table that share a Display name (In our case it was Email Address 1), Canvas apps add the logical name in parentheses after it into the display name, so that the user can differentiate.

- When marshalling the AttributeTypeCode.Lookup, you can check if the expression requires the whole object or not. For example: If I'm adding a property with a display name 'Parent Account', I can check if the original expression contains an Ident token with the same name. If not, I can say that the FormulaType of this is just TableEmpty, which will be the correct type but no additional metadata. If we have this Ident token in the expression, we can recursively download the Table add it (You will find the related Table name in the targets property). Now, your expression will have access the metadata of the two required tables.

![tokenSplit](/uploads/2023/08/using-powerfx-with-dataverse1.png)

- The expression in Power Fx is made to be as user friendly as possible. That's why, when writing it, we use display names of tables and fields or a specific culture (There may be a difference between using a comma or a dot for decimal numbers). When moving this from environment to environment however, we will need to transform the expression to be as neutral as possible. For this, you can use the `CheckResult.ApplyGetInvariant()` method, which will do exactly that.
