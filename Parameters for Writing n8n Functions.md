# Parameters for Writing n8n Functions

**Tags**: #n8n #workflow #automation #code-node #javascript #data-transformation #reference

## Overview

This document serves as a reference for writing JavaScript functions in n8n Code nodes. It covers the available variables, data structures, and best practices for creating effective data transformations and manipulations within n8n workflows.

## Built-in Variables

### Current Node Input

| Variable | Description | Available in Code Node? |
|----------|-------------|-------------------------|
| `$binary` | Shorthand for `$input.item.binary`. Incoming binary data from a node. | ❌ (Use `$input.item.binary` directly in Code nodes) |
| `$input.all()` | All input items in current node. | ✅ |
| `$input.first()` | First input item in current node. | ✅ |
| `$input.last()` | Last input item in current node. | ✅ |
| `$input.params` | Object containing the query settings of the previous node. | ✅ |
| `$json` | Shorthand for `$input.item.json`. Incoming JSON data from a node. | ✅ (when running once for each item) |
| `$input.context.noItemsLeft` | Boolean indicating if the node is still processing items. | ✅ |

### Output of Other Nodes

| Variable | Description | Available in Code Node? |
|----------|-------------|-------------------------|
| `$("<node-name>").all(branchIndex?, runIndex?)` | Returns all items from a given node. | ✅ |
| `$("<node-name>").first(branchIndex?, runIndex?)` | The first item output by the given node. | ✅ |
| `$("<node-name>").last(branchIndex?, runIndex?)` | The last item output by the given node. | ✅ |
| `$("<node-name>").item` | The linked item. | ❌ (Use `$("<node-name>").first()` or `$("<node-name>").all()[index]` instead) |
| `$("<node-name>").params` | Object containing the query settings of the given node. | ✅ |

### Object Functions

| Function | Description |
|----------|-------------|
| `isEmpty()` | Checks if the Object has no key-value pairs. |
| `merge(object: Object)` | Merges two Objects into a single Object. |
| `hasField(fieldName: String)` | Checks if the Object has a given field. |

## Item Linking in n8n

In n8n, "item linking" refers to the relationship between items across different nodes in a workflow. While the `$("<node-name>").item` method exists in expressions, it's not available in Code nodes. This is because:

1. Code nodes operate on a different execution model than expressions
2. The direct item linking context isn't preserved in the same way within Code nodes

### When to Use Which Method

Instead of using `.item`, use these alternatives based on your specific needs:

| Method | When to Use | Example |
|--------|-------------|---------|
| `$("<node-name>").first()` | When you need only the first item from another node, regardless of which item is being processed in the current node | `const userData = $("HTTP Request").first().json;` |
| `$("<node-name>").all()[index]` | When you need a specific item by index from another node | `const thirdUser = $("HTTP Request").all()[2].json;` |
| `$("<node-name>").all()` | When you need to process all items from another node | `const allUsers = $("HTTP Request").all().map(item => item.json);` |

This approach provides the same functionality while working within the Code node's execution context. Choose the method that best matches your data access pattern to optimize both code clarity and performance.

## Data Structure in n8n

In n8n, data flows between nodes in a standardized structure:

```javascript
{
  "json": {
    // The main data object containing the node's output
    // This is what you access with $json
    "property1": "value1",
    "property2": "value2",
    // ...
  },
  "binary": {
    // Binary data if present
  }
}
```

When a previous node sends data to a Code node, you can access it using:
- `$json` to access the JSON data directly
- `$input.item.json` for the same data (full form)
- `$input.item.binary` for any binary data

## Working with Binary Data in n8n

While the shorthand `$binary` isn't available in Code nodes, you can still work with binary data using the full form `$input.item.binary`. Binary data in n8n is typically represented as an object with properties for each binary field:

```javascript
{
  "fieldName1": {
    "data": "base64-encoded-data",
    "mimeType": "application/pdf",
    "fileName": "document.pdf",
    "fileSize": 12345
  },
  "fieldName2": {
    // Another binary field
  }
}
```

In this structure:
- The top-level keys (like "fieldName1") are the names of your binary fields
- Each binary field contains metadata and the actual base64-encoded data

### Example: Processing Binary Data

```javascript
// Check if binary data exists
if (!$input.item.binary || !$input.item.binary.binaryFile) {
  throw new Error("No binary data found");
}

// Access binary data properties
const binaryField = $input.item.binary.binaryFile;  // Get the specific binary field
const fileName = binaryField.fileName;
const mimeType = binaryField.mimeType;
const fileSize = binaryField.fileSize;
const base64Data = binaryField.data;  // The actual base64-encoded content

// Return both JSON and binary data
return {
  json: {
    fileName,
    mimeType,
    fileSize,
    processingResult: "success"
  },
  binary: $input.item.binary // Pass through the binary data
};
```

### Example: Creating Binary Data

```javascript
// Create new binary data
const newBinaryData = {
  binaryFile: {  // Field name for the binary data
    data: Buffer.from("Hello World").toString("base64"),  // The actual base64-encoded content
    mimeType: "text/plain",
    fileName: "hello.txt",
    fileSize: 11
  }
};

// Return with both JSON and new binary data
return {
  json: { status: "success" },
  binary: newBinaryData
};
```

## Common Patterns for Code Nodes

### 1. Basic Data Transformation

```javascript
// Input validation
if (!$json.someRequiredField) {
  throw new Error("Missing required field");
}

// Transform data
const transformed = {
  newField1: $json.oldField1.toUpperCase(),
  newField2: $json.oldField2 * 2,
  // ...
};

// Return the transformed data
return transformed;
```

### 2. Processing Arrays

```javascript
// Ensure we have an array
if (!Array.isArray($json.items)) {
  throw new Error("Expected items to be an array");
}

// Transform each item
const transformedItems = $json.items.map(item => {
  return {
    id: item.id,
    name: item.name.trim(),
    // ...
  };
});

// Filter if needed
const filteredItems = transformedItems.filter(item => item.name !== "");

return filteredItems;
```

### 3. Accessing Data from Other Nodes

```javascript
// Get data from another node by name
const otherNodeData = $("OtherNodeName").all();

// Combine with current data
return {
  currentData: $json,
  otherData: otherNodeData[0].json
};
```

## Best Practices

1. **Always validate input data** before processing to avoid runtime errors.
2. **Handle multiple input formats** to make your code robust against workflow changes.
3. **Use try/catch blocks** for error handling, especially when parsing JSON or performing operations that might fail.
4. **Implement per-record error handling** to prevent a single bad record from failing the entire operation.
5. **Add comprehensive logging** to help diagnose issues without re-running the workflow.
6. **Return data in the expected format** for the next node in the workflow.
7. **Add comments** to explain complex transformations or logic.
8. **Break down complex operations** into smaller, reusable functions.
9. **Use meaningful variable names** to make the code more readable.
10. **Consider performance** for large datasets by using efficient operations.
11. **Handle field name variations** (camelCase, PascalCase, snake_case) for more robust code.

## Example: Robust Data Transformation

```javascript
// Get data from input with flexible format handling
let parsedData;

// Log input data structure for debugging
console.log("Input data type:", typeof $json);
console.log("Input data structure:", JSON.stringify($json).substring(0, 200) + "...");

// Check multiple possible data locations
if ($json.body?.resultObject) {
  const rawData = $json.body.resultObject;
  
  // Handle string or object format
  if (typeof rawData === 'string') {
    try {
      parsedData = JSON.parse(rawData);
    } catch (error) {
      throw new Error("Failed to parse resultObject: " + error.message);
    }
  } else {
    parsedData = rawData;
  }
} else if (Array.isArray($json)) {
  parsedData = $json;
} else if ($json.data && Array.isArray($json.data)) {
  parsedData = $json.data;
} else if ($json.items && Array.isArray($json.items)) {
  parsedData = $json.items;
} else if ($json.resultObject) {
  // Handle resultObject directly (not in body)
  if (typeof $json.resultObject === 'string') {
    try {
      parsedData = JSON.parse($json.resultObject);
    } catch (error) {
      throw new Error("Failed to parse resultObject: " + error.message);
    }
  } else if (Array.isArray($json.resultObject)) {
    parsedData = $json.resultObject;
  }
} else if (typeof $json === 'object' && Object.keys($json).length > 0) {
  // Single object case - wrap in array
  parsedData = [$json];
} else {
  throw new Error("No data found in input. Expected an array or object with data.");
}

// Ensure it's an array
if (!Array.isArray(parsedData)) {
  throw new Error("Parsed data is not an array");
}

// Process each item with error handling for individual records
const result = parsedData.map((item, index) => {
  try {
    // Handle field name variations (camelCase, PascalCase, snake_case)
    const id = item.id || item.ID || item.Id || "";
    const name = item.name || item.Name || item.NAME || "";
    const email = item.email || item.Email || item.EMAIL || "";
    
    // Process and return the item
    return {
      id: id,
      name: name ? name.trim() : "",
      email: email ? email.toLowerCase() : "",
      // Add more fields as needed
    };
  } catch (error) {
    console.log(`Error processing record ${index}:`, error.message);
    // Return a minimal record with error information
    return {
      id: item.id || `error-record-${index}`,
      name: "Error Processing Record",
      errorMessage: error.message
    };
  }
});

// Return the processed data
return result;
```

## Debugging Tips

1. Use `console.log()` to output debug information (visible in the execution log).
2. Test your code with sample data before running it on production data.
3. Break down complex transformations into steps and verify each step.
4. Use the n8n debugger to inspect the data at each node.

## Input Format Handling Strategies

When working with n8n Code nodes, one of the most challenging aspects is handling different input formats. Here are strategies to make your code more robust:

### 1. Flexible Input Detection

```javascript
// Comprehensive input format detection
let data;

// Try multiple possible input formats
if (Array.isArray($json)) {
  // Direct array input
  data = $json;
} 
else if ($json.body?.resultObject) {
  // Nested in body.resultObject (common in HTTP responses)
  const rawData = $json.body.resultObject;
  data = typeof rawData === 'string' ? JSON.parse(rawData) : rawData;
}
else if ($json.data && Array.isArray($json.data)) {
  // In a data property
  data = $json.data;
}
else if ($input && $input.first() && $input.first().json) {
  // In $input.first().json
  data = $input.first().json;
}
else {
  // Fallback to using $json directly
  data = [$json];
}
```

### 2. Diagnostic Logging

```javascript
// Log input structure for debugging
console.log("Input type:", typeof $json);
console.log("Input keys:", Object.keys($json));
console.log("Input structure:", JSON.stringify($json).substring(0, 200) + "...");

// Log which format was detected
console.log(`Processing ${data.length} records`);
```

### 3. Per-Record Error Handling

```javascript
// Process each record with individual error handling
const results = data.map((item, index) => {
  try {
    // Process the item
    return {
      id: item.id,
      // other fields
    };
  } catch (error) {
    console.log(`Error processing record ${index}:`, error.message);
    // Return a minimal record with error information
    return {
      id: item.id || `error-record-${index}`,
      errorMessage: error.message
    };
  }
});
```

## Performance Optimization

When working with large datasets in n8n, performance becomes critical. Here are strategies to optimize your Code node execution:

### 1. Minimize Iterations

```javascript
// Instead of multiple iterations
const filtered = data.filter(item => item.active);
const mapped = filtered.map(item => ({ id: item.id, name: item.name }));

// Combine operations in a single iteration
const result = data.reduce((acc, item) => {
  if (item.active) {
    acc.push({ id: item.id, name: item.name });
  }
  return acc;
}, []);
```

### 2. Use Early Returns

```javascript
// Process data conditionally with early returns
function processItem(item) {
  // Skip processing if conditions aren't met
  if (!item.shouldProcess) return null;
  if (item.isArchived) return null;
  
  // Only process items that pass the conditions
  return { /* processed item */ };
}

const results = data
  .map(processItem)
  .filter(Boolean); // Remove null results
```

### 3. Lazy Evaluation

```javascript
// Instead of processing everything upfront
const allItems = $("DataSource").all().map(item => {
  // Complex processing for ALL items
  return processItem(item.json);
});

// Process only what you need when you need it
function getItemById(id) {
  const items = $("DataSource").all();
  const item = items.find(i => i.json.id === id);
  if (!item) return null;
  
  // Only process the specific item you need
  return processItem(item.json);
}
```

### 4. Batch Processing

For very large datasets, consider processing in batches:

```javascript
// Process in batches of 100
const allItems = $("LargeDataSource").all();
const batchSize = 100;
const results = [];

for (let i = 0; i < allItems.length; i += batchSize) {
  const batch = allItems.slice(i, i + batchSize);
  
  // Process this batch
  const processedBatch = batch.map(item => {
    // Process each item
    return { /* processed item */ };
  });
  
  results.push(...processedBatch);
  
  // Optional: Log progress
  console.log(`Processed ${Math.min(i + batchSize, allItems.length)} of ${allItems.length} items`);
}
```

## Common Issues & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| **"No data found in input"** <br>`Error: No data found in input` | The input data structure doesn't match what your code expects | ```javascript<br>// Log the input structure<br>console.log("Input type:", typeof $json);<br>console.log("Input structure:", JSON.stringify($json).substring(0, 200) + "...");<br><br>// Implement flexible input detection<br>``` |
| **"Cannot read property 'X' of undefined"** <br>`TypeError: Cannot read property 'name' of undefined` | Trying to access a property on an undefined object | ```javascript<br>// Instead of this (will fail)<br>const name = $json.user.name;<br><br>// Use optional chaining<br>const name = $json.user?.name || "Unknown";<br><br>// Or check existence first<br>const name = ($json.user && $json.user.name) ? $json.user.name : "Unknown";<br>``` |
| **"X is not a function"** <br>`TypeError: $json.items.map is not a function` | Trying to use array methods on non-array data | ```javascript<br>// Always check if it's an array first<br>const items = Array.isArray($json.items) ? $json.items : [];<br>const mappedItems = items.map(item => /* ... */);<br>``` |
| **"Unexpected token in JSON"** <br>`SyntaxError: Unexpected token < in JSON at position 0` | Trying to parse invalid JSON, often HTML or XML returned from an API | ```javascript<br>try {<br>  const data = JSON.parse(rawData);<br>  // Process data<br>} catch (error) {<br>  // Log the raw data<br>  console.log("Failed to parse JSON. Raw data:", rawData);<br>  throw new Error(`JSON parsing failed: ${error.message}`);<br>}<br>``` |
| **"Cannot access property of null"** <br>`TypeError: Cannot access property 'length' of null` | Trying to access properties on null values | ```javascript<br>// Instead of this<br>const count = $json.results.length;<br><br>// Check for null/undefined first<br>const count = $json.results ? $json.results.length : 0;<br>``` |
| **"Node not found"** <br>`Error: Node "PreviousNode" not found` | Referencing a node that doesn't exist or has been renamed | ```javascript<br>try {<br>  const data = $("PreviousNode").first();<br>  // Process data<br>} catch (error) {<br>  // Fallback to current input<br>  console.log("Could not access PreviousNode");<br>  const data = $input.first();<br>}<br>``` |

---

This reference document was created to provide guidance on writing effective n8n functions. Refer to the [official n8n documentation](https://docs.n8n.io/code/builtin/overview/) for the most up-to-date information.
