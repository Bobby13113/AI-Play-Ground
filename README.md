<p align="center">
  <a href="" rel="noopener">
 <img width=200px height=200px src="https://avatars.githubusercontent.com/u/181013883?v=4" alt="Project logo"></a>
</p>

<h3 align="center">Project Title</h3>

<div align="center">

[![Status](https://img.shields.io/badge/status-active-success.svg)]()
[![GitHub Issues](https://img.shields.io/github/issues/kylelobo/The-Documentation-Compendium.svg)](https://github.com/kylelobo/The-Documentation-Compendium/issues)
[![GitHub Pull Requests](https://img.shields.io/github/issues-pr/kylelobo/The-Documentation-Compendium.svg)](https://github.com/kylelobo/The-Documentation-Compendium/pulls)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](/LICENSE)

</div>

---

<p align="center"> .
    <br> 
</p>

### 🔧 Core Integrations

| Tool           | Purpose                                                              |
| -------------- | -------------------------------------------------------------------- |
| **ComfyUI**    | Node-based image generation with custom workflows and model control  |
| **Open-WebUI** | LLM interface for prompt engineering, chat, and agent-based tasks    |
| **N8N**        | Visual workflow automation for chaining AI tasks, APIs, and triggers |

## **Table of Contents**

- [Text to Image Generation Setup](#setup)
  - [Requirements](#requirements)
  - [Installation](#installation)
  - [Running the Server](#running-the-server)
  - [Running with Docker](#running-with-docker)
- [Functionality](#functionality)
  - [Positive and Negative Prompts](#positive-and-negative-prompts)
  - [LLM-Assisted Prompt Generation](#llm-assisted-prompt-generation)
  - [Quick Prompts](#quick-prompts)
  - [Image Caching and Navigation](#image-caching-and-navigation)
  - [UI Reset](#ui-reset)
- [Architecture](#architecture)
  - [Backend](#backend)
    - [Key Endpoints](#key-endpoints)
  - [Frontend](#frontend)
    - [UI Components](#ui-components)
  - [Tools and Libraries](#tools-and-libraries)
- [Testing](#testing)

---

## **Setup**

### **Requirements**

1. **Conda Environment**:

   - The project uses Conda for environment management. Make sure Conda is installed on your system.

2. **ComfyUI**:

   - ComfyUI is installed and running on an Unbuntu VM. You must have the checkpoint you wish to use installed in the `checkpoints` folder for the workflow.
   - Alternatively, you can modify `workflow.json` to use any other model/checkpoint. I will have several different workflow.json files avaiable for diffenert image projects.

3. **Ollama**:

   - Ollama LLM server is installed on a Unbuntu VM on the Dell R730 with the Nvidia A10 configured with PCI Passthrough.

4. **Configuration via `.env`**:
   - The project uses a `.env` file for configuring server addresses. Below are custom configuration settings:
     ```bash
     COMFYUI_SERVER_ADDRESS=192.168.50.50:8080
     OLLAMA_SERVER_ADDRESS=192.168.50.32:11434
     OLLAMA_MODEL=gemma3:12b
     ```
   - Adjust these values to match your environment.

### **Installation**

1. **Clone the Repository**:

   ```bash
   cd /root
   git clone https://github.com/Bobby13113/ImageGen.git
   cd ImageGen
   ```

2. **Set Up Conda Environment**:

   Create and activate the Conda environment:

```bash
cd
mkdir conda
cd conda
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-aarch64.sh
bash ./Miniconda3-latest-Linux-aarch64.sh
```

```bash
conda create --name bobslab python=3.12
conda activate bobslab
```

3. **Install Dependencies**:

   Install the project dependencies listed in `requirements.txt`:

   ```bash
   pip install -r requirements.txt
   ```

4. **Set Up `.env` File**:

   Ensure the `.env` file exists in the project root and contains the correct server addresses for ComfyUI and Ollama.

   ```bash
     COMFYUI_SERVER_ADDRESS=192.168.50.50:8080
     OLLAMA_SERVER_ADDRESS=192.168.50.32:11434
     OLLAMA_MODEL=gemma3:12b
   ```

---

## **Running the Server**

### **Local Environment**

To run the FastAPI server in your local environment, use the following command:

```bash
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

This will start the app on `http://localhost:8000/`.

To ensure that ComfyUI is functioning correctly, you can test the connection using the workflow defined in `workflow.json`.

## **FastAPI Documentation**

To access the API documents you can use this url or add /redoc or /docs to the applications home url.

```bash
http://192.168.50.72:8000/redoc
```

---

## **Running with Docker**

If you prefer to run the application inside a Docker container, the following steps guide you through building and running the containerized application.

### **1. Build the Docker Image**

Navigate to the project directory and build the Docker image:

```bash
docker build -t text-2-imagelab .
```

### **2. Run the Docker Container**

Once the Docker image is built, run the container:

```bash
docker run -d -p 8000:8000 --env-file .env --name text-2-imagelab text-2-imagelab
```

This command will:

- Start the container in **detached mode** (`-d`).
- Map port **8000** of the container to port **8000** on your host.
- Use the `.env` file to set environment variables.

### **3. Access the Application**

You can now access the application at `http://localhost:8000/`

---

## **Functionality**

### **Positive and Negative Prompts**

- **Positive Prompt**: Specifies the elements to include in the generated image (e.g., "4k, highly detailed, hyperrealistic").
- **Negative Prompt**: Specifies elements to avoid in the image (e.g., "blurry, watermark").

### **LLM-Assisted Prompt Generation**

- **Ask LLM for a Creative Idea**: The user can request a creative prompt suggestion from a locally hosted LLM (Ollama). The generated prompt can be applied to the positive prompt field.

### **Quick Prompts**

- **Preconfigured Prompts**: Both positive and negative quick prompts are available via buttons. Clicking a button auto-fills the corresponding input field.
- **Custom Prompts**: Themed prompts (like **Halloween** or **Christmas**) are dynamically loaded from the `quick_prompts.json` file. Adding new themes is as easy as editing this file.

### **Image Caching and Navigation**

- **Image History**: The app caches generated images within the session. Users can navigate through cached images using the **Previous** and **Next** buttons.
- **Cache Clearing**: Cached images are cleared when the browser is refreshed or when the **Reset** button is clicked.

### **UI Reset**

- The **Reset** button clears all input fields, resets generated images, and clears the image cache.

---

## **Architecture**

### **Backend**

The backend is powered by **FastAPI** and handles the following operations:

- Generating images using ComfyUI.
- Fetching creative suggestions from the local LLM.
- Serving quick prompts from configuration files.

#### **Key Endpoints**

1. **POST /generate_images/**

   - **Description**: Generates an AI image using the provided prompts and image settings.
   - **Request Example**:
     ```json
     {
       "positive_prompt": "a beautiful sunset",
       "negative_prompt": "blurry",
       "steps": 25,
       "width": 512,
       "height": 512
     }
     ```
   - **Response**: A binary stream containing the generated image.

2. **POST /ask_llm/**

   - **Description**: Requests a creative prompt from the local LLM server (Ollama).
   - **Request Example**:
     ```json
     {
       "positive_prompt": "a beautiful sunset"
     }
     ```
   - **Response**:
     ```json
     {
       "assistant_reply": "How about a stunning sunset over the mountains with golden light reflecting on the water?"
     }
     ```

3. **GET /quick_prompts/**
   - **Description**: Retrieves quick prompt configurations from the `quick_prompts.json` file for dynamic UI updates.
   - **Response Example**:
     ```json
     {
       "Positive Quick Prompts": [
         { "label": "4K", "value": "4K" },
         { "label": "Highly Detailed", "value": "highly detailed" }
       ],
       "Negative Quick Prompts": [
         { "label": "Blurry", "value": "blurry" },
         { "label": "Watermark", "value": "watermark" }
       ],
       "Halloween": [
         { "label": "Black Background", "value": "black background" },
         { "label": "Witch", "value": "witch" }
       ]
     }
     ```

### **Frontend**

The frontend is built with HTML, CSS, and JavaScript. It dynamically interacts with the backend for:

- Generating images.
- Fetching creative prompts from the LLM.
- Loading quick prompt configurations from `quick_prompts.json`.

#### **UI Components**

1. **Image Generation Form**:

   - Includes fields for positive and negative prompts, image steps, width, and height.
   - Quick prompt buttons for easy input.

2. **LLM Integration**:

   - A section that allows users to request and apply creative prompts generated by the LLM.

3. **Image Display and Navigation**:

   - Displays the generated images and includes buttons for navigating through cached images.

4. **Reset Functionality**:
   - A **Reset** button to clear all input fields and generated image history.

### **Tools and Libraries**

1. **FastAPI**: Web framework for building the backend.
2. **Uvicorn**: ASGI server used to run the FastAPI application.
3. **Ollama**: Locally hosted LLM for generating creative prompts.
4. **Pillow**: Python Imaging Library used to handle image operations.
5. **Bootstrap**: CSS framework for styling the UI.
6. **JavaScript (Fetch API)**: Handles asynchronous requests to the backend.

---

## **Testing**

You can test the ComfyUI workflow by running the FastAPI server as described above. Use the `/generate_images/` endpoint to generate images and validate that the workflow is functioning correctly.

To test the LLM integration, use the `/ask_llm/` endpoint to request a creative prompt from the locally hosted Ollama LLM.

For Docker-based setups, ensure that the `.env` file is correctly set up with the server addresses and run the container as described in the [Running with Docker](#running-with-docker) section.

## **Kill Server**

If you need to force kill the server process, you can use the following command:

```bash
sudo kill -9 $(sudo lsof -t -i :8000)
```

## ✍️ Authors <a name = "authors"></a>

- [@Bobby13113](https://github.com/Bobby13113) - Idea & Initial work

## 🎉 Acknowledgements <a name = "acknowledgement"></a>

- Hat tip to anyone whose code was used
- Inspiration
- References
