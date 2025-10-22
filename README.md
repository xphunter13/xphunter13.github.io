<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>AI Image Editor</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link
    href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap"
    rel="stylesheet"
  />
  <style>
    body {
      font-family: "Inter", sans-serif;
      background-color: #1e293b;
      color: #e2e8f0;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
    }
    .card {
      background-color: #334155;
      box-shadow: 0 10px 15px rgba(0, 0, 0, 0.2);
    }
    .btn-primary {
      transition: all 0.2s;
    }
    .btn-primary:hover {
      transform: translateY(-1px);
      box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    }
    .image-container {
      min-height: 250px;
      background-color: #475569;
      border: 2px dashed #64748b;
      display: flex;
      align-items: center;
      justify-content: center;
      overflow: hidden;
      position: relative;
    }
    .image-container img {
      object-fit: contain;
      width: 100%;
      height: 100%;
    }
    .spinner {
      border: 4px solid rgba(255, 255, 255, 0.1);
      border-top: 4px solid #3b82f6;
      border-radius: 50%;
      width: 40px;
      height: 40px;
      animation: spin 1s linear infinite;
    }
    @keyframes spin {
      0% {
        transform: rotate(0deg);
      }
      100% {
        transform: rotate(360deg);
      }
    }
  </style>
</head>
<body class="p-4 sm:p-8">
  <!-- Header -->
  <header class="text-center mb-8">
    <h1 class="text-3xl font-bold text-white mb-2">AI Image Command Center</h1>
    <p class="text-slate-400">
      Upload an image and tell the AI exactly what you want to change.
    </p>
  </header>

  <!-- Main Editor Grid -->
  <main class="max-w-6xl mx-auto w-full grid grid-cols-1 lg:grid-cols-3 gap-8">
    <!-- Control Panel -->
    <div class="lg:col-span-1 card p-6 rounded-xl space-y-6">
      <h2 class="text-xl font-semibold mb-4 text-white">
        1. Input Image & Prompt
      </h2>

      <!-- Image Upload -->
      <div>
        <label
          for="imageUpload"
          class="block text-sm font-medium mb-2 text-slate-300"
          >Upload Original Image (JPG/PNG)</label
        >
        <input
          type="file"
          id="imageUpload"
          accept="image/jpeg, image/png"
          class="block w-full text-sm text-slate-500
            file:mr-4 file:py-2 file:px-4
            file:rounded-full file:border-0
            file:text-sm file:font-semibold
            file:bg-indigo-50 file:text-indigo-700
            hover:file:bg-indigo-100"
        />
      </div>

      <!-- Original Image Preview (with drag & drop) -->
      <div
        class="h-48 image-container rounded-lg"
        id="originalImagePreview"
        title="Drag & Drop an image here"
      >
        <span class="text-slate-400 text-center p-4"
          >Image will appear here (or drag & drop).</span
        >
      </div>

      <!-- Prompt Input -->
      <div>
        <label
          for="editPrompt"
          class="block text-sm font-medium mb-2 text-slate-300"
          >Editing Instruction Prompt</label
        >
        <textarea
          id="editPrompt"
          rows="4"
          class="w-full rounded-lg bg-slate-600 text-white p-3 border border-slate-500 focus:ring-blue-500 focus:border-blue-500"
          placeholder="e.g., Change the background to a neon cyberpunk city. Add sunglasses to the person."
        ></textarea>
      </div>

      <!-- Generate Button -->
      <button
        id="generateBtn"
        class="btn-primary w-full py-3 px-4 bg-indigo-600 text-white font-semibold rounded-lg shadow-md hover:bg-indigo-700 disabled:opacity-50 flex items-center justify-center"
        disabled
      >
        <span id="btnText">Generate Edited Image</span>
        <div id="loadingSpinner" class="spinner ml-2 hidden"></div>
      </button>

      <!-- Error Message -->
      <div
        id="errorMessage"
        class="hidden text-sm p-3 rounded-lg bg-red-800 text-red-100"
      ></div>
    </div>

    <!-- Results Display -->
    <div class="lg:col-span-2 card p-6 rounded-xl">
      <h2 class="text-xl font-semibold mb-4 text-white">
        2. AI Generated Result
      </h2>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-4 h-full">
        <!-- Original Image -->
        <div class="flex flex-col">
          <h3 class="text-lg font-medium mb-2 text-slate-300">Original</h3>
          <div class="image-container flex-grow rounded-lg p-2" id="originalDisplay">
            <img
              id="originalImg"
              src="https://placehold.co/400x300/475569/cbd5e1?text=Upload+Image+to+Start"
              alt="Original Image"
              class="rounded-lg max-h-full"
            />
          </div>
        </div>

        <!-- Generated Image -->
        <div class="flex flex-col">
          <h3 class="text-lg font-medium mb-2 text-blue-400">
            Generated (AI Edit)
          </h3>
          <div
            class="image-container flex-grow rounded-lg p-2"
            id="generatedDisplay"
          >
            <img
              id="generatedImg"
              src="https://placehold.co/400x300/1e293b/94a3b8?text=Generated+Result+Here"
              alt="AI Generated Image"
              class="rounded-lg max-h-full"
            />
          </div>

          <!-- Download Button -->
          <button
            id="downloadBtn"
            class="mt-4 py-2 px-3 bg-blue-500 text-white rounded-lg hover:bg-blue-600 hidden"
          >
            Download Edited Image
          </button>
        </div>
      </div>

      <!-- Text Response -->
      <div
        id="textResponse"
        class="mt-6 p-4 rounded-lg bg-slate-600 hidden text-sm text-slate-200"
      ></div>

      <!-- Citations -->
      <div id="citationPanel" class="mt-6 p-4 rounded-lg bg-slate-600 hidden">
        <h3 class="text-sm font-semibold text-slate-300 mb-2">
          Sources (Grounding)
        </h3>
        <ul
          id="sourceList"
          class="text-xs text-slate-400 list-disc list-inside"
        ></ul>
      </div>
    </div>
  </main>

  <!-- JavaScript Logic -->
  <script>
    const imageUpload = document.getElementById("imageUpload");
    const originalImagePreview = document.getElementById("originalImagePreview");
    const originalImg = document.getElementById("originalImg");
    const generatedImg = document.getElementById("generatedImg");
    const editPrompt = document.getElementById("editPrompt");
    const generateBtn = document.getElementById("generateBtn");
    const btnText = document.getElementById("btnText");
    const loadingSpinner = document.getElementById("loadingSpinner");
    const errorMessage = document.getElementById("errorMessage");
    const citationPanel = document.getElementById("citationPanel");
    const sourceList = document.getElementById("sourceList");
    const downloadBtn = document.getElementById("downloadBtn");
    const textResponse = document.getElementById("textResponse");

    let uploadedImageBase64 = null;
    const apiKey = ""; // Insert your Gemini API key here

    function displayError(message) {
      errorMessage.textContent = message;
      errorMessage.classList.remove("hidden");
      console.error("ERROR:", message);
    }
    function clearError() {
      errorMessage.classList.add("hidden");
      errorMessage.textContent = "";
    }
    function setLoading(isLoading) {
      generateBtn.disabled = isLoading || uploadedImageBase64 === null;
      if (isLoading) {
        btnText.textContent = "Generating...";
        loadingSpinner.classList.remove("hidden");
        clearError();
        generatedImg.src =
          "https://placehold.co/400x300/1e293b/94a3b8?text=Generating+Image...";
        downloadBtn.classList.add("hidden");
        textResponse.classList.add("hidden");
      } else {
        btnText.textContent = "Generate Edited Image";
        loadingSpinner.classList.add("hidden");
      }
    }

    // --- File Upload ---
    function handleFile(file) {
      if (file.size > 5 * 1024 * 1024) {
        displayError("The file size is too large. Please upload an image under 5MB.");
        uploadedImageBase64 = null;
        generateBtn.disabled = true;
        return;
      }
      const reader = new FileReader();
      reader.onload = (e) => {
        const base64 = e.target.result.split(",")[1];
        const mimeType = file.type;
        if (!mimeType.startsWith("image/")) {
          displayError("Invalid file type. Please upload a JPEG or PNG image.");
          return;
        }
        uploadedImageBase64 = { data: base64, mimeType };
        const imgUrl = e.target.result;
        originalImagePreview.innerHTML = `<img src="${imgUrl}" class="rounded-lg object-contain w-full h-full" />`;
        originalImg.src = imgUrl;
        generateBtn.disabled = false;
        clearError();
      };
      reader.onerror = () => displayError("Failed to read file.");
      reader.readAsDataURL(file);
    }

    imageUpload.addEventListener("change", (e) => {
      if (e.target.files[0]) handleFile(e.target.files[0]);
    });

    // --- Drag & Drop Support ---
    originalImagePreview.addEventListener("dragover", (e) => e.preventDefault());
    originalImagePreview.addEventListener("drop", (e) => {
      e.preventDefault();
      const file = e.dataTransfer.files[0];
      if (file) handleFile(file);
    });

    // --- Gemini API ---
    async function callGeminiAPI(payload, maxRetries = 5) {
      const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image-preview:generateContent?key=${apiKey}`;
      for (let attempt = 0; attempt < maxRetries; attempt++) {
        try {
          const response = await fetch(apiUrl, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(payload),
          });
          if (response.ok) return await response.json();
          if (response.status === 429 && attempt < maxRetries - 1) {
            await new Promise((r) => setTimeout(r, Math.pow(2, attempt) * 1000));
            continue;
          }
          throw new Error(`API error: ${response.status}`);
        } catch (err) {
          if (attempt === maxRetries - 1) throw err;
          await new Promise((r) => setTimeout(r, Math.pow(2, attempt) * 1000));
        }
      }
    }

    async function generateImageEdit() {
      const userPrompt = editPrompt.value.trim();
      if (!uploadedImageBase64) return displayError("Please upload an image first.");
      if (!userPrompt) return displayError("Please enter a prompt.");
      setLoading(true);

      const payload = {
        contents: [
          {
            parts: [
              { text: userPrompt },
              { inlineData: uploadedImageBase64 },
            ],
          },
        ],
        generationConfig: { responseModalities: ["TEXT", "IMAGE"] },
      };

      try {
        const result = await callGeminiAPI(payload);
        const candidate = result.candidates?.[0];
        if (!candidate) throw new Error("No result from API");

        const imagePart = candidate.content.parts.find(
          (p) => p.inlineData && p.inlineData.mimeType.startsWith("image/")
        );
        const textPart = candidate.content.parts.find((p) => p.text);

        if (imagePart?.inlineData?.data) {
          generatedImg.src = `data:${imagePart.inlineData.mimeType};base64,${imagePart.inlineData.data}`;
          downloadBtn.classList.remove("hidden");
        }

        if (textPart?.text) {
          textResponse.textContent = textPart.text;
          textResponse.classList.remove("hidden");
        }

        citationPanel.classList.add("hidden");
      } catch (err) {
        displayError("Image generation failed: " + err.message);
        generatedImg.src =
          "https://placehold.co/400x300/800000/ffffff?text=ERROR";
      } finally {
        setLoading(false);
      }
    }

    generateBtn.addEventListener("click", generateImageEdit);
    editPrompt.addEventListener("input", () => {
      generateBtn.disabled =
        !uploadedImageBase64 || editPrompt.value.trim() === "";
    });

    // --- Download Edited Image ---
    downloadBtn.addEventListener("click", () => {
      const link = document.createElement("a");
      link.href = generatedImg.src;
      link.download = "ai_edited_image.png";
      link.click();
    });

    generateBtn.disabled = true;
  </script>
</body>
</html>
