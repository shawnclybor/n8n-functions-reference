**n8n Code Node Reference & AI-Assisted Function Writing Guide**

Overview

This repository provides a comprehensive reference for writing JavaScript functions in n8n Code nodes. It includes:
	•	A breakdown of built-in variables available in n8n.
	•	Best practices for data transformations and manipulations within workflows.
	•	Guidance on handling JSON and binary data in n8n.
	•	Debugging strategies to troubleshoot errors efficiently.
	•	Performance optimizations to ensure smooth execution of functions.

This reference is designed for both beginners and experienced users looking to streamline their n8n automation workflows by leveraging custom JavaScript functions.

How to Use This File

Option 1: AI-Assisted Function Writing (Recommended)

Leverage an LLM (Large Language Model) such as ChatGPT or Claude to assist with writing your n8n functions:
	1.	Upload this file to ChatGPT or another LLM.
	2.	Describe your use case in detail.
	•	Include your node’s input data.
	•	Describe the expected output in detail.
	•	(Optional but highly recommended) Provide a sample expected output to guide the AI.
	3.	Ask the LLM to generate a JavaScript function based on your requirements.
	4.	Copy the generated function into your n8n Code node and run a test.
	5.	If an error occurs, copy the error message back into the LLM and ask it to debug the function.
	6.	Iterate until the function executes successfully.

This method allows for rapid function prototyping and error resolution.

Option 2: Using the Cline Extension with VS Code

For users who prefer a more dynamic code-writing experience, you can use this file with VS Code and the Cline extension.

Steps to Use Cline for n8n Code Writing:
	1.	Install the Cline extension in VS Code.
	2.	Open this file in VS Code as a reference.
	3.	Use Cline to interact with an AI model directly inside VS Code.
	4.	Write and refine your n8n JavaScript functions dynamically.
	5.	Test your functions in n8n, iterating as needed.

This approach allows for faster testing and refining, minimizing back-and-forth copy-pasting between applications.

Troubleshooting & Debugging

If your function throws an error:
	•	Copy the error message and paste it into ChatGPT (or another LLM).
	•	Ask it to analyze and fix the issue.
	•	Test the revised function and repeat if needed.

Common debugging strategies include:
✅ Checking input data structure.
✅ Handling undefined values properly.
✅ Using console.log() for in-node debugging.
✅ Validating JSON parsing and array manipulations.
✅ Ensuring proper item linking within the workflow.

For common error fixes and performance optimization tips, refer to the “Best Practices & Debugging” section in this document.

Why Use This File?
	•	✅ Saves Time – No need to manually look up n8n’s documentation every time.
	•	✅ AI-Assisted Coding – Quickly generate, debug, and optimize functions with LLMs.
	•	✅ Flexible Workflow – Use in ChatGPT, VS Code, or directly in n8n.
	•	✅ Improves Automation – Helps refine complex data transformations and API handling.

Contributing

If you have improvements or additional insights:
	1.	Fork the repository.
	2.	Make changes and submit a PR.
	3.	Share useful AI-generated function examples to improve this reference.

License

This reference is open-source and can be freely used, modified, and shared.

Let me know if you’d like any tweaks!
