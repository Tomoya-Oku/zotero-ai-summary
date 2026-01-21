# LLM-based Paper Summarization Plugin for Zotero

This plugin automatically summarizes papers added to Zotero and saves the generated summaries as notes.
Original: https://github.com/cs-qyzhang/zotero-ai-summary

## Workflow

This tool is implemented as a script for [zotero-actions-tags](https://github.com/windingwind/zotero-actions-tags).

1. A new paper is added to Zotero  
2. zotero-actions-tags triggers a JavaScript script  
3. The script retrieves the PDF file path and paper title  
4. The PDF is sent to a server for parsing and chunking  
5. An LLM API is called locally to summarize the paper  
6. The generated Markdown is sent to the server and converted to HTML  
7. The HTML summary is written to the paper’s note in Zotero  

## Deployment | Setup

### How to install Zotero on Linux (Ubuntu 22.04)
See [zotero-deb](https://github.com/retorquere/zotero-deb/).

### Install zotero-actions-tags

Install the [zotero-actions-tags](https://github.com/windingwind/zotero-actions-tags) plugin and configure it as shown below.
<img width="1142" height="790" alt="image" src="https://github.com/user-attachments/assets/9ac71534-fdc2-4cdb-a712-87bf2a51f4c3" />
<img width="485" height="577" alt="image" src="https://github.com/user-attachments/assets/ae3f7c82-f9d4-4656-9e32-085ae7447d7a" />

### Configuration

#### Important Notes

- If you use [ZotMoov](https://github.com/wileyyugioh/zotmoov) or  
  [ZotFile](https://github.com/jlegewie/zotfile) to store PDFs in a synced drive
  such as OneDrive, **set `only_link_file` to `true`**.
- Configure **`openaiBaseUrl` / `modelName` / `apiKey`** according to the LLM API you use.
- Adjust **`chunkSize`** according to the context length of the selected model.
- You may translate **`stuffPrompt` / `mapPrompt` / `reducePrompt`** into your preferred language,  
  but **do not modify `{title}` and `{text}`**.

#### TLDR

- Using ZotMoov / ZotFile with “Link to File” → `only_link_file = true`
- Configure `openaiBaseUrl`, `modelName`, `apiKey` for your LLM provider
- Adjust `chunkSize` to fit the model’s context length
- Prompts can be translated, but `{title}` and `{text}` must remain unchanged

## Configuration Parameters (`zotero_script.js`)

At the top of `zotero_script.js`, the following configuration options are defined:

```js
let serverUrl = "https://paper_summarizer.jianyue.tech";
let only_link_file = false;
let timeout = 30;
let openaiBaseUrl = "https://dashscope.aliyuncs.com/compatible-mode/v1";
// Gemini
// let openaiBaseUrl = "https://generativelanguage.googleapis.com/v1beta/openai/";
let modelName = "qwen-plus-latest";
let apiKey = "sk-xxxxxxxxxxxxx";
let chunkSize = 64000;
let chunkOverlap = 1000;
let stuffPrompt = "";
let mapPrompt = "";
let reducePrompt = "";
```

- serverUrl: The server URL for parsing PDF files and converting the summarized markdown into HTML. Defaults to the author's public server. If you need to summarize sensitive files, it's recommended to deploy your own server; simply clone this repository and run python server.py.
- only_link_file: Use this in conjunction with ZotMoov or ZotFile. If you save PDF papers in Zotero as "Link to File" using these (or similar) plugins, set only_link_file to true; otherwise, set it to false.
- timeout: Also used with ZotMoov or ZotFile. This parameter specifies the maximum number of seconds to wait after adding a new paper before checking whether the PDF download is complete.
- openaiBaseUrl: The API endpoint compatible with OpenAI. The specific URL depends on the model provider you are using; by default, it is Qwen, see https://www.aliyun.com/product/bailian.
- modelName: The model name provided when calling the API.
- apiKey: The API key for the LLM.
- chunkSize: The model's context size. PDF documents that exceed the context window must be split into multiple segments and summarized using a map-reduce approach.
- chunkOverlap: The overlap size between segments when using the map-reduce approach.
- stuffPrompt: The prompt used when the entire PDF document fits within the model’s context window.
- mapPrompt: The prompt used during the map phase of the map-reduce approach.
- reducePrompt: The prompt used during the reduce phase of the map-reduce approach.
