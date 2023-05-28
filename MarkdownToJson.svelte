<script>
    let markdownInput = "";
    let jsonOutput = "";

    function markdownToJson(markdown) {
        let lines = markdown.split('\n');
        let json = [];
        let currentCategory = null;
        let currentGroup = null;
        let currentItem = null;

        lines.forEach(line => {
            if (line.startsWith('# ')) {
                currentCategory = {
                    categoryName: line.slice(2).trim(),
                    groups: []
                };
                json.push(currentCategory);
            } else if (line.startsWith('## ')) {
                currentGroup = {
                    group: line.slice(3).trim(),
                    items: []
                };
                currentCategory.groups.push(currentGroup);
            } else if (line.startsWith('### ')) {
                currentItem = {
                    item: line.slice(4).trim(),
                    description: "",
                    details: {}
                };
                currentGroup.items.push(currentItem);
            } else if (line.startsWith('- ')) {
                currentItem.description = line.slice(2).trim();
            } else if (line.startsWith('#### ')) {
                let key = line.slice(5).trim();
                currentItem.details[key] = '';
            } else {
                let lastKey = Object.keys(currentItem.details).pop();
                currentItem.details[lastKey] += line.trim();
            }
        });

        return json;
    }

    function convertToJson() {
        let json = markdownToJson(markdownInput);
        jsonOutput = JSON.stringify(json, null, 2);
    }
</script>

<style>
    textarea {
        width: 100%;
        height: 200px;
    }
</style>

<div class="markdown-to-json-converter">
    <h1>Markdown to JSON Converter</h1>
    <textarea bind:value={markdownInput} placeholder="Enter your markdown here..."></textarea>
    <button on:click={convertToJson}>Convert to JSON</button>
    <textarea bind:value={jsonOutput} placeholder="Your converted JSON will appear here..."></textarea>
</div>
