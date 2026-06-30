---
layout: post
title: "Extending Sitecore TreeList with Queryable Data Sources and Validation"
date: 2026-06-30 11:00:00 -0600
categories: [sitecore, csharp, custom-controls]
tags: [sitecore, dotnet, custom-field-types, content-editor]
excerpt: "A production-ready extension of Sitecore's TreeList field that supports query-based data sources, template filtering, and validation warnings—solving real-world content authoring challenges."
---

# Extending Sitecore TreeList with Queryable Data Sources and Validation

So here's the thing about Sitecore's out-of-the-box TreeList fields—they're great until they're not. And on this one enterprise project, they definitely weren't cutting it.

I needed dynamic queries, template filtering, and validation warnings. The kind of stuff that makes content authors' lives easier and prevents 3am "why is this broken?" Slack messages.

<p><br></p>

## Standing on Shoulders

Credit where it's due: [Jammy Kam wrote a killer extension](https://jammykam.wordpress.com/2016/01/06/specifying-query-and-parameters-for-sitecore-treelist-field-source/) back in 2016 that handled queryable TreeLists. I was using it on a multi-site Sitecore instance and kept hitting edge cases.

So I forked his approach and extended it. Here's what I added and why.

<p><br></p>
<p><br></p>

## The Problem

Picture this: Multi-site Sitecore instance. Content authors need to pick related articles, but only from their current site. And only certain content types should even show up in the picker.

The vanilla TreeList? Nope. You'd have to either:
- Hardcode every single path (maintenance nightmare)
- Create template variations for each site (also a nightmare)
- Write custom code every time (you know where this is going)

And here's the kicker—when content gets moved around (and it always does), authors would select items that were technically invalid. No warnings, no feedback. Just silent failures waiting to explode in production.

### What Was Breaking

1. **Static only** — Couldn't use Sitecore queries to figure out the tree root dynamically
2. **Template filtering was a mess** — No way to say "show these types, but only let them select those types"
3. **Zero validation** — Authors had no clue when they picked something that was gonna cause problems later
4. **GUID case sensitivity** — Because nothing says "fun Friday afternoon" like debugging why `{abc-123}` doesn't match `{ABC-123}`

<p><br></p>

## The Fix

I extended the base TreeList control and added four things that made life way better:

### 1. Query-Based Resolution

Now you can use actual Sitecore queries to figure out where the tree root is:

```csharp
// Dynamic resolution based on current item
DataSource=query:./ancestor-or-self::*[@@templateid='{GUID}']/*[@@name='Content']&IncludeTemplatesForSelection={GUID}
```

No more hardcoded paths. The query runs in context of the current item, so it adapts automatically.

<p><br></p>

### 2. Separate Display vs. Selection Filtering

Sometimes you want authors to see certain items in the tree (for context), but not be able to actually select them.

- **IncludeTemplatesForDisplay** — What shows up in the tree
- **IncludeTemplatesForSelection** — What they can actually pick

Game changer for complex content hierarchies.

<p><br></p>

### 3. Validation Warnings

Here's my favorite part: If an author has selected something that's no longer valid (because content moved, or the query changed), they get a visual warning.

The field appends `[Not in selection list]` right there in the Content Editor. No more mystery bugs six months later.

<p><br></p>

### 4. Template ID Normalization

That GUID case sensitivity bug? Fixed. All template IDs get normalized to uppercase before the query runs.

Turns out `{abc-123}` and `{ABC-123}` shouldn't be different things. Who knew?

<p><br></p>
<p><br></p>


## How It Actually Works

Okay, let's get into the weeds.

### The Three Overrides

I'm hooking into three methods from the base `TreeList`:

1. **Source Property** — Parses the custom parameters (display templates, selection templates, etc.)
2. **DataSource Property** — This is where the magic happens. Executes the query and figures out the tree root
3. **GetHeaderValue** — Injects the validation warnings when rendering selected items

Nothing crazy, just strategic override points.

<p><br></p>

### Query Resolution (The Fun Part)

```csharp
if (rawSource.StartsWith("query:", StringComparison.OrdinalIgnoreCase))
{
    var current = Sitecore.Context.ContentDatabase.GetItem(base.ItemID);
    
    // Normalize Template IDs to uppercase
    queryPart = Regex.Replace(
        queryPart,
        @"\{[0-9a-fA-F\-]{36}\}",
        m => m.Value.ToUpperInvariant()
    );
    
    // Execute query
    obj = LookupSources.GetItems(current, queryPart).FirstOrDefault();
    
    return obj?.Paths.FullPath ?? string.Empty;
}
```

<p><br></p>

**Why this works:**
- Query runs in the context of wherever the author is editing—so it's always relative and dynamic
- The GUID normalization? That's the regex on line 117. Saved me so many headaches
- If the query bombs or returns nothing, we just bail gracefully. Field still renders, no explosions

<p><br></p>

### Validation Logic

```csharp
private bool IsItemInSelectionList(Item item)
{
    var dataSourcePath = this.DataSource;
    var rootItem = Sitecore.Context.ContentDatabase?.GetItem(dataSourcePath);
    
    // Item must be the root or a descendant
    return item.ID == rootItem.ID || 
           item.Paths.FullPath.StartsWith(rootItem.Paths.FullPath, 
               StringComparison.OrdinalIgnoreCase);
}
```

<p><br></p>

**What this gives you:**
- Authors see broken stuff immediately—no surprises during QA
- When content gets moved (and it will), you'll know about it
- Zero silent failures. If something's wrong, it screams at you in the UI

<p><br></p>

### Error Handling (Don't Skip This)

```csharp
try
{
    obj = LookupSources.GetItems(current, queryPart).FirstOrDefault();
}
catch (Exception ex)
{
    logger?.LogError("QueryableTreeList field failed to execute query. Exception: {Exception}", ex);
}
```

**Production reality:**
- Malformed queries? Logged. You'll see them in your logs, not in production at midnight
- Field still renders even if the query explodes. Authors can keep working
- No exceptions bubble up to users. It just... handles it

<p><br></p>
<p><br></p>

## Setting It Up

### Step 1: Register the Field Type

Head to `/sitecore/system/Field types/List Types/` and create a new field type.

Fill in:
- **Name:** QueryableTreeList
- **Assembly:** YourNamespace.Foundation.SitecoreCMS
- **Class:** YourNamespace.Foundation.SitecoreCMS.Custom.Controls.QueryableTreeList
- **Control:** YourNamespace:QueryableTreeList

Replace `YourNamespace` with your actual namespace (obviously).

<p><br></p>

### Step 2: Use It

In your template's field source, throw in a query like this:

```
DataSource=query:./ancestor-or-self::*[@@templateid='{SITE-TEMPLATE-ID}']/*[@@name='Articles']&IncludeTemplatesForSelection={ARTICLE-TEMPLATE-ID}
```

Swap in your actual template GUIDs. The query syntax is standard Sitecore query—if you've written them before, you're good.

<p><br></p>

### Step 3: Test It

1. Create an item with your template
2. Pop it open in Content Editor
3. Check that the TreeList shows the right root (based on the query)
4. Try selecting stuff. Move items around. Verify the warnings show up when they should

<p><br></p>
<p><br></p>

## The Trade-offs (Because Nothing's Free)

### Performance

Real talk: This runs a query every time the field renders in Content Editor. If you've got deep tree structures or complex queries, it might get sluggish.

**What I did about it:** Used it strategically. Not every TreeList needs this. Pick your battles. You could also cache query results if performance becomes an issue.

### Learning Curve

Your content architects need to know Sitecore query syntax. Not everyone does.

**What helped:** Documented the common patterns. Created a few field source templates they could copy-paste. After that, they were good.

### Debugging

Malformed queries fail silently by design (see the error handling above). This is intentional—I'd rather have a field that doesn't populate than one that crashes the Content Editor.

**But:** You need good logging. When queries fail, you'll want to know why. The logger calls saved me multiple times.

<p><br></p>
<p><br></p>

## What Happened After Shipping It

Deployed this to a big multi-site Sitecore instance. Here's what I learned:

**Support tickets dropped 40%** — Authors were catching their own mistakes before publishing. The validation warnings did exactly what they were supposed to do.

**Templates became reusable** — No more creating variations for every site. One template, dynamic queries, done.

**The GUID normalization thing** — Eliminated a whole category of "why is my list empty?" bugs. Those were a pain to debug before.

**Logging saved my ass** — Multiple times. When someone wrote a bad query, I could see exactly what failed and why. Essential for multi-site setups.

<p><br></p>
<p><br></p>

## The Code

Grab it from my [GitHub repo](https://github.com/marioarce/marioarce/blob/main/docs/code-samples/sitecore/QueryableTreeList.cs). It's de-branded and ready to drop into your project.

What you get:
- ✅ Tested in production (the real test)
- ✅ Error handling that doesn't explode
- ✅ Comments that actually explain why, not what
- ✅ Works with Sitecore 10+ (maybe earlier, haven't tested)

<p><br></p>
<p><br></p>

## Final Thoughts

Look, custom field types aren't always the answer. But when the OOTB stuff doesn't fit? Don't hack around it. Extend it properly.

This QueryableTreeList solved real problems for real authors. The validation warnings alone paid for the development time within a month.

**The lesson:** Invest in author experience. Happy authors = better content = fewer 3am production issues. Simple math.

<p><br></p>
<p><br></p>

## Related Resources

- [Original inspiration by Jammy Kam](https://jammykam.wordpress.com/2016/01/06/specifying-query-and-parameters-for-sitecore-treelist-field-source/)
- [Sitecore Query Syntax Documentation](https://doc.sitecore.com/xp/en/developers/latest/platform-administration-and-architecture/sitecore-query.html)
- [Custom Field Types in Sitecore](https://doc.sitecore.com/xp/en/developers/latest/sitecore-experience-manager/custom-field-types.html)



