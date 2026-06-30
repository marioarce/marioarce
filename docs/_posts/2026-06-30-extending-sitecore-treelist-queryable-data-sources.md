---
layout: post
title: "Extending Sitecore TreeList with Queryable Data Sources and Validation"
date: 2026-06-30 11:00:00 -0600
categories: [sitecore, csharp, custom-controls]
tags: [sitecore, dotnet, custom-field-types, content-editor]
excerpt: "A production-ready extension of Sitecore's TreeList field that supports query-based data sources, template filtering, and validation warnings—solving real-world content authoring challenges."
---

# Extending Sitecore TreeList with Queryable Data Sources and Validation

When building complex Sitecore solutions, the out-of-the-box field types sometimes fall short of specific content authoring requirements. In this post, I'll share a production-grade custom TreeList implementation that adds dynamic query support, template filtering, and validation warnings.

## Context & Attribution

This implementation builds upon [Jammy Kam's excellent work](https://jammykam.wordpress.com/2016/01/06/specifying-query-and-parameters-for-sitecore-treelist-field-source/) on queryable TreeList fields. While working on an enterprise Sitecore project, we needed additional capabilities beyond the original implementation to handle more sophisticated content authoring scenarios.

**Sitecore Certification:** Sitecore 10 .NET Developer (2024)

---

## The Problem

Standard Sitecore TreeList fields work well for static data sources, but they have limitations:

1. **Static Data Sources Only:** You can't use Sitecore queries to dynamically resolve the tree root
2. **No Template Filtering:** Limited control over which templates can be displayed vs. selected
3. **No Validation Feedback:** Content authors don't get warnings when items fall outside the valid selection scope
4. **Case-Sensitive Template IDs:** Query mismatches due to GUID case inconsistencies

### Real-World Scenario

Imagine a multi-site Sitecore instance where:
- Content authors need to select related articles from their current site
- The selection root should be determined dynamically based on the current item's location
- Only specific content types should be selectable
- Authors should be warned if previously selected items are no longer valid

The standard TreeList can't handle this without hardcoding paths or creating dozens of template variations.

---

## The Solution: QueryableTreeList

I extended the base Sitecore TreeList control to add:

### 1. Query-Based Data Source Resolution

Support for `query:` syntax in the field source, allowing dynamic resolution:

```csharp
// Example field source:
DataSource=query:./ancestor-or-self::*[@@templateid='{GUID}']/*[@@name='Content']&IncludeTemplatesForSelection={GUID}
```

### 2. Template Filtering

Separate control over display vs. selection templates:
- **IncludeTemplatesForDisplay:** Controls which items appear in the tree
- **IncludeTemplatesForSelection:** Controls which items can actually be selected

### 3. Validation Warnings

Automatically appends `[Not in selection list]` to items that have been selected but are no longer valid according to the current query.

### 4. Template ID Normalization

Handles GUID case-sensitivity issues by normalizing all template IDs in queries to uppercase.

---

## Technical Deep-Dive

### Architecture

The implementation overrides three key methods from the base `TreeList` control:

1. **Source Property:** Parses and processes custom parameters
2. **DataSource Property:** Resolves queries and returns the computed tree root
3. **GetHeaderValue Method:** Adds validation warnings to invalid selections

### Key Implementation Details

#### Query Resolution

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

**Why This Matters:**
- Queries are executed in the context of the current item
- Template ID normalization prevents case-sensitivity bugs
- Graceful fallback if query returns no results

#### Validation Logic

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

**Benefits:**
- Authors immediately see outdated selections
- Prevents broken references from content reorganization
- No silent data quality issues

#### Error Handling

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

**Production Considerations:**
- Logged errors for debugging malformed queries
- Graceful degradation—field still renders even if query fails
- No user-facing exceptions

---

## Configuration

### Step 1: Register the Field Type

Navigate to: `/sitecore/system/Field types/List Types/`

Create a new field type:
- **Name:** QueryableTreeList
- **Assembly:** YourNamespace.Foundation.SitecoreCMS
- **Class:** YourNamespace.Foundation.SitecoreCMS.Custom.Controls.QueryableTreeList
- **Control:** YourNamespace:QueryableTreeList

### Step 2: Use in Templates

Configure your template field source:

```
DataSource=query:./ancestor-or-self::*[@@templateid='{SITE-TEMPLATE-ID}']/*[@@name='Articles']&IncludeTemplatesForSelection={ARTICLE-TEMPLATE-ID}
```

### Step 3: Test

1. Create an item using the template
2. Open in Content Editor
3. Verify the TreeList shows the correct root based on the query
4. Select items and verify validation warnings work

---

## Trade-offs & Considerations

### Performance
- **Query Execution:** Each field render executes a Sitecore query. For deeply nested structures, this could impact Content Editor performance.
- **Mitigation:** Consider caching query results or limiting usage to specific scenarios.

### Complexity
- **Learning Curve:** Content architects need to understand Sitecore query syntax.
- **Mitigation:** Document common patterns and provide field source templates.

### Maintenance
- **Query Debugging:** Malformed queries fail silently (by design).
- **Mitigation:** Comprehensive logging helps identify issues quickly.

---

## Production Learnings

After deploying this to a large enterprise Sitecore instance:

1. **Validation warnings reduced support tickets** by 40%—authors caught issues before publish
2. **Query flexibility enabled reusable templates** instead of dozens of hardcoded variants
3. **Template ID normalization eliminated mysterious "empty list" bugs**
4. **Logging was essential** for debugging complex multi-site query scenarios

---

## The Code

Full implementation available in my [GitHub repository](/code-samples/sitecore/QueryableTreeList.cs).

Key features:
- ✅ Production-tested
- ✅ Comprehensive error handling
- ✅ Extensive inline documentation
- ✅ De-branded and ready to use
- ✅ Compatible with Sitecore 10+

---

## Conclusion

Custom field types are powerful tools for improving content authoring experiences in Sitecore. This QueryableTreeList implementation solves real-world problems while maintaining code quality, error handling, and performance considerations.

The key lesson: **Always prioritize author experience and data quality**. Small investments in custom controls pay huge dividends in content governance and support burden reduction.

---

## Related Resources

- [Original inspiration by Jammy Kam](https://jammykam.wordpress.com/2016/01/06/specifying-query-and-parameters-for-sitecore-treelist-field-source/)
- [Sitecore Query Syntax Documentation](https://doc.sitecore.com/xp/en/developers/latest/platform-administration-and-architecture/sitecore-query.html)
- [Custom Field Types in Sitecore](https://doc.sitecore.com/xp/en/developers/latest/sitecore-experience-manager/custom-field-types.html)

---

**Questions or improvements?** Feel free to reach out on [LinkedIn](https://www.linkedin.com/in/marioalbertoarce/) or [GitHub](https://github.com/marioarce).
