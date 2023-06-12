<template>
    <div>
        <!-- Form Builder Interface -->
        <div>
            <h2>Form Builder</h2>
            <div v-for="(field, index) in formFields" :key="index">
                <label>Label:</label>
                <input v-model="field.label" type="text" />

                <label>Type:</label>
                <select v-model="field.type">
                    <option value="text">Text</option>
                    <option value="checkbox">Checkbox</option>
                    <option value="textarea">Textarea</option>
                    <option value="select">Dropdown Select</option>
                </select>

                <label v-if="field.type === 'select'">Options:</label>
                <textarea
                    v-if="field.type === 'select'"
                    v-model="field.options"
                    placeholder="Enter options, separated by commas"
                ></textarea>
            </div>
            <button @click="addField">Add Field</button>
        </div>

        <!-- Form Preview -->
        <div>
            <h2>Form Preview</h2>
            <div v-for="(field, index) in formFields" :key="index">
                <label>{{ field.label }}:</label>
                <span v-if="field.type === 'checkbox'">{{
                    field.value ? "Checked" : "Unchecked"
                }}</span>
                <span v-if="field.type === 'text'">
                    <input type="text" :placeholder="field.label" />
                </span>
                <span v-if="field.type === 'textarea'">
                    <textarea type="text" :placeholder="field.label"></textarea>
                </span>
                <span v-else>{{ field.value }}</span>
            </div>
            <button @click="copyFormHtml">Copy Form HTML</button>
        </div>

        <!-- Chat Interface -->
        <div>
            <h2>Chat Interface</h2>
            <div class="chat-container">
                <div class="chat-messages">
                    <div v-for="(message, index) in chatMessages" :key="index">
                        <div
                            v-if="message.role === 'user'"
                            class="user-message"
                        >
                            {{ message.content }}
                        </div>
                        <div v-else class="system-message">
                            {{ message.content }}
                        </div>
                    </div>
                </div>
                <div class="chat-input">
                    <input
                        v-model="inputMessage"
                        type="text"
                        placeholder="Type your message"
                    />
                    <button @click="sendMessage">Send</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from "axios";

export default {
    data() {
        return {
            formFields: [
                { label: "Name", type: "text", value: "" },
                { label: "Email", type: "text", value: "" },
            ],
            chatMessages: [],
            inputMessage: "",
            chatGptApiKey:
                "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",
        };
    },

    methods: {
        addField() {
            const newField = {
                label: "New Field",
                type: "text",
                value: "",
                options: "",
            };
            this.formFields.push(newField);
        },

        copyFormHtml() {
            const formHtml = this.generateFormHtml();
            this.copyToClipboard(formHtml);
        },

        generateFormHtml() {
            let formHtml = "";

            for (const field of this.formFields) {
                formHtml += `<div>`;
                formHtml += `<label>${field.label}:</label>`;

                if (field.type === "checkbox") {
                    formHtml += `<input type="checkbox" ${
                        field.value ? "checked" : ""
                    } disabled>`;
                } else if (field.type === "textarea") {
                    formHtml += `<textarea readonly>${field.value}</textarea>`;
                } else if (field.type === "select") {
                    const options = field.options
                        .split(",")
                        .map((option) => option.trim());
                    formHtml += `<select disabled>`;
                    for (const option of options) {
                        formHtml += `<option ${
                            field.value === option ? "selected" : ""
                        }>${option}</option>`;
                    }
                    formHtml += `</select>`;
                } else {
                    formHtml += `<span>${field.value}</span>`;
                }

                formHtml += `</div>`;
            }

            return formHtml;
        },

        copyToClipboard(text) {
            const textarea = document.createElement("textarea");
            textarea.value = text;
            document.body.appendChild(textarea);
            textarea.select();
            document.execCommand("copy");
            document.body.removeChild(textarea);
            alert("Form HTML copied to clipboard!");
        },

        sendMessage() {
            const userMessage = {
                role: "user",
                content: this.inputMessage,
            };
            this.chatMessages.push(userMessage);
            this.fetchChatResponse(this.inputMessage);
            this.inputMessage = "";
        },

        async fetchChatResponse(input) {
            try {
                const response = await axios.post(
                    "https://api.openai.com/v1/chat/completions",
                    {
                        prompt: input,
                        max_tokens: 50,
                    },
                    {
                        headers: {
                            Authorization: `Bearer ${this.chatGptApiKey}`,
                            "Content-Type": "application/json",
                        },
                    }
                );

                const chatGptResponseContent =
                    response.data.choices[0].text.trim();

                const chatGptResponse = {
                    role: "system",
                    content: chatGptResponseContent,
                };

                this.chatMessages.push(chatGptResponse);
            } catch (error) {
                console.error("Failed to fetch ChatGPT response:", error);
            }
        },
    },
};
</script>

<style scoped>
/* Styles for the chat container and messages */
.chat-container {
    border: 1px solid #ccc;
    padding: 10px;
    margin-top: 20px;
}

.chat-messages {
    max-height: 200px;
    overflow-y: auto;
}

.user-message {
    margin-bottom: 10px;
    background-color: #f0f0f0;
    padding: 5px;
}

.system-message {
    margin-bottom: 10px;
    font-style: italic;
    color: #666;
}

/* Styles for the form preview */
button {
    margin-top: 10px;
}
</style>
