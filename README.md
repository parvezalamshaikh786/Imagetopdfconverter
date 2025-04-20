<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Image to PDF Converter</title>
    <!-- jsPDF library for PDF generation -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <!-- html2canvas for rendering images -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <style>
        :root {
            --primary-color: #4a6bff;
            --secondary-color: #f8f9fa;
            --text-color: #333;
            --bg-color: #fff;
            --border-color: #ddd;
            --error-color: #ff4757;
            --success-color: #2ed573;
        }

        [data-theme="dark"] {
            --primary-color: #6c8eff;
            --secondary-color: #2d3436;
            --text-color: #f5f6fa;
            --bg-color: #1e272e;
            --border-color: #555;
            --error-color: #ff6b81;
            --success-color: #7bed9f;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            transition: background-color 0.3s, color 0.3s;
        }

        body {
            background-color: var(--bg-color);
            color: var(--text-color);
            line-height: 1.6;
            padding: 20px;
            min-height: 100vh;
        }

        .container {
            max-width: 900px;
            margin: 0 auto;
            padding: 20px;
        }

        h1 {
            text-align: center;
            margin-bottom: 30px;
            color: var(--primary-color);
        }

        .upload-area {
            border: 2px dashed var(--border-color);
            border-radius: 10px;
            padding: 30px;
            text-align: center;
            margin-bottom: 20px;
            cursor: pointer;
            background-color: var(--secondary-color);
        }

        .upload-area:hover {
            border-color: var(--primary-color);
        }

        .upload-area.active {
            border-color: var(--primary-color);
            background-color: rgba(74, 107, 255, 0.05);
        }

        .upload-icon {
            font-size: 50px;
            color: var(--primary-color);
            margin-bottom: 10px;
        }

        .file-input {
            display: none;
        }

        .btn {
            background-color: var(--primary-color);
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            margin: 5px;
            transition: opacity 0.3s;
        }

        .btn:hover {
            opacity: 0.9;
        }

        .btn:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }

        .btn-secondary {
            background-color: var(--secondary-color);
            color: var(--text-color);
            border: 1px solid var(--border-color);
        }

        .preview-container {
            margin-top: 30px;
        }

        .preview-title {
            margin-bottom: 15px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .image-list {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
            gap: 15px;
            margin-bottom: 20px;
        }

        .image-item {
            position: relative;
            border-radius: 5px;
            overflow: hidden;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
            background-color: var(--secondary-color);
        }

        .image-item img {
            width: 100%;
            height: 150px;
            object-fit: contain;
            display: block;
            background-color: white;
        }

        .image-actions {
            position: absolute;
            top: 5px;
            right: 5px;
            display: flex;
        }

        .action-btn {
            background-color: rgba(255, 0, 0, 0.7);
            color: white;
            border: none;
            width: 25px;
            height: 25px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            margin-left: 5px;
            font-size: 12px;
        }

        .action-btn:hover {
            background-color: rgba(0, 0, 0, 0.9);
        }

        .settings {
            background-color: var(--secondary-color);
            padding: 15px;
            border-radius: 10px;
            margin-bottom: 20px;
        }

        .settings-title {
            margin-bottom: 10px;
            display: flex;
            align-items: center;
            justify-content: space-between;
        }

        .settings-options {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
            gap: 15px;
        }

        .form-group {
            margin-bottom: 10px;
        }

        label {
            display: block;
            margin-bottom: 5px;
            font-weight: 500;
        }

        select, input {
            width: 100%;
            padding: 8px;
            border-radius: 5px;
            border: 1px solid var(--border-color);
            background-color: var(--bg-color);
            color: var(--text-color);
        }

        .loading {
            display: none;
            text-align: center;
            margin: 20px 0;
        }

        .spinner {
            border: 4px solid rgba(0, 0, 0, 0.1);
            border-radius: 50%;
            border-top: 4px solid var(--primary-color);
            width: 30px;
            height: 30px;
            animation: spin 1s linear infinite;
            margin: 0 auto 10px;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .message {
            padding: 10px;
            border-radius: 5px;
            margin-bottom: 15px;
            text-align: center;
            display: none;
        }

        .error {
            background-color: rgba(255, 71, 87, 0.1);
            color: var(--error-color);
            border: 1px solid var(--error-color);
        }

        .success {
            background-color: rgba(46, 213, 115, 0.1);
            color: var(--success-color);
            border: 1px solid var(--success-color);
        }

        .theme-toggle {
            background: none;
            border: none;
            font-size: 24px;
            cursor: pointer;
            color: var(--text-color);
        }

        .controls {
            display: flex;
            justify-content: center;
            flex-wrap: wrap;
            margin-top: 20px;
        }

        @media (max-width: 768px) {
            .image-list {
                grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
            }
            
            .settings-options {
                grid-template-columns: 1fr;
            }
        }

        @media (max-width: 480px) {
            .container {
                padding: 10px;
            }
            
            .upload-area {
                padding: 20px 10px;
            }
            
            .image-list {
                grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Image to PDF Converter</h1>
        
        <button class="theme-toggle" id="themeToggle">🌓</button>
        
        <!-- Upload Area -->
        <div class="upload-area" id="uploadArea">
            <div class="upload-icon">📁</div>
            <h3>Drag & Drop Images Here</h3>
            <p>or</p>
            <button class="btn" id="selectFilesBtn">Select Images</button>
            <input type="file" id="fileInput" class="file-input" accept="image/*" multiple>
        </div>
        
        <!-- Messages -->
        <div class="message error" id="errorMessage"></div>
        <div class="message success" id="successMessage"></div>
        
        <!-- PDF Settings -->
        <div class="settings">
            <div class="settings-title">
                <h3>PDF Settings</h3>
            </div>
            <div class="settings-options">
                <div class="form-group">
                    <label for="pageSize">Page Size</label>
                    <select id="pageSize">
                        <option value="a4">A4</option>
                        <option value="letter">Letter</option>
                        <option value="legal">Legal</option>
                    </select>
                </div>
                <div class="form-group">
                    <label for="orientation">Orientation</label>
                    <select id="orientation">
                        <option value="portrait">Portrait</option>
                        <option value="landscape">Landscape</option>
                    </select>
                </div>
                <div class="form-group">
                    <label for="margin">Margin (mm)</label>
                    <input type="number" id="margin" min="0" max="50" value="10">
                </div>
                <div class="form-group">
                    <label for="imageFit">Image Fit</label>
                    <select id="imageFit">
                        <option value="contain">Contain (fit whole image)</option>
                        <option value="cover">Cover (fill page)</option>
                    </select>
                </div>
            </div>
        </div>
        
        <!-- Image Preview Section -->
        <div class="preview-container" id="previewContainer" style="display: none;">
            <div class="preview-title">
                <h3>Selected Images</h3>
                <button class="btn btn-secondary" id="clearAllBtn">Clear All</button>
            </div>
            <div class="image-list" id="imageList"></div>
        </div>
        
        <!-- Loading Indicator -->
        <div class="loading" id="loadingIndicator">
            <div class="spinner"></div>
            <p>Generating PDF...</p>
        </div>
        
        <!-- Controls -->
        <div class="controls">
            <button class="btn" id="generatePdfBtn" disabled>Generate PDF</button>
        </div>
    </div>

    <script>
        // Initialize jsPDF
        const { jsPDF } = window.jspdf;
        
        // DOM Elements
        const uploadArea = document.getElementById('uploadArea');
        const fileInput = document.getElementById('fileInput');
        const selectFilesBtn = document.getElementById('selectFilesBtn');
        const imageList = document.getElementById('imageList');
        const previewContainer = document.getElementById('previewContainer');
        const generatePdfBtn = document.getElementById('generatePdfBtn');
        const clearAllBtn = document.getElementById('clearAllBtn');
        const errorMessage = document.getElementById('errorMessage');
        const successMessage = document.getElementById('successMessage');
        const loadingIndicator = document.getElementById('loadingIndicator');
        const themeToggle = document.getElementById('themeToggle');
        
        // PDF Settings Elements
        const pageSizeSelect = document.getElementById('pageSize');
        const orientationSelect = document.getElementById('orientation');
        const marginInput = document.getElementById('margin');
        const imageFitSelect = document.getElementById('imageFit');
        
        // Store uploaded images
        let uploadedImages = [];
        
        // Initialize the app
        function init() {
            setupEventListeners();
            checkThemePreference();
        }
        
        // Set up event listeners
        function setupEventListeners() {
            // File selection
            selectFilesBtn.addEventListener('click', () => fileInput.click());
            fileInput.addEventListener('change', handleFileSelect);
            
            // Drag and drop
            uploadArea.addEventListener('dragover', (e) => {
                e.preventDefault();
                uploadArea.classList.add('active');
            });
            
            uploadArea.addEventListener('dragleave', () => {
                uploadArea.classList.remove('active');
            });
            
            uploadArea.addEventListener('drop', (e) => {
                e.preventDefault();
                uploadArea.classList.remove('active');
                if (e.dataTransfer.files.length) {
                    fileInput.files = e.dataTransfer.files;
                    handleFileSelect({ target: fileInput });
                }
            });
            
            // PDF generation
            generatePdfBtn.addEventListener('click', generatePdf);
            
            // Clear all images
            clearAllBtn.addEventListener('click', clearAllImages);
            
            // Theme toggle
            themeToggle.addEventListener('click', toggleTheme);
        }
        
        // Handle file selection
        function handleFileSelect(event) {
            const files = event.target.files;
            if (!files || files.length === 0) return;
            
            // Reset messages
            hideMessage(errorMessage);
            hideMessage(successMessage);
            
            // Check file types
            const validImageTypes = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];
            const invalidFiles = Array.from(files).filter(
                file => !validImageTypes.includes(file.type)
            );
            
            if (invalidFiles.length > 0) {
                showMessage(errorMessage, `Invalid file type(s): ${invalidFiles.map(f => f.name).join(', ')}. Please upload only images (JPEG, PNG, GIF, WEBP).`);
                return;
            }
            
            // Process valid files
            Array.from(files).forEach(file => {
                if (validImageTypes.includes(file.type)) {
                    const reader = new FileReader();
                    reader.onload = (e) => {
                        const img = new Image();
                        img.onload = () => {
                            uploadedImages.push({
                                file,
                                url: e.target.result,
                                width: img.width,
                                height: img.height
                            });
                            updateImageList();
                        };
                        img.src = e.target.result;
                    };
                    reader.readAsDataURL(file);
                }
            });
            
            // Reset file input to allow selecting the same files again
            fileInput.value = '';
        }
        
        // Update the image list display
        function updateImageList() {
            if (uploadedImages.length === 0) {
                previewContainer.style.display = 'none';
                generatePdfBtn.disabled = true;
                return;
            }
            
            previewContainer.style.display = 'block';
            generatePdfBtn.disabled = false;
            
            imageList.innerHTML = '';
            
            uploadedImages.forEach((image, index) => {
                const imageItem = document.createElement('div');
                imageItem.className = 'image-item';
                
                const img = document.createElement('img');
                img.src = image.url;
                img.alt = image.file.name;
                
                const actions = document.createElement('div');
                actions.className = 'image-actions';
                
                const deleteBtn = document.createElement('button');
                deleteBtn.className = 'action-btn';
                deleteBtn.innerHTML = '×';
                deleteBtn.title = 'Remove image';
                deleteBtn.addEventListener('click', (e) => {
                    e.stopPropagation();
                    removeImage(index);
                });
                
                // Only show move buttons if there's more than one image
                if (uploadedImages.length > 1) {
                    const moveUpBtn = document.createElement('button');
                    moveUpBtn.className = 'action-btn';
                    moveUpBtn.innerHTML = '↑';
                    moveUpBtn.title = 'Move up';
                    moveUpBtn.addEventListener('click', (e) => {
                        e.stopPropagation();
                        moveImage(index, 'up');
                    });
                    
                    const moveDownBtn = document.createElement('button');
                    moveDownBtn.className = 'action-btn';
                    moveDownBtn.innerHTML = '↓';
                    moveDownBtn.title = 'Move down';
                    moveDownBtn.addEventListener('click', (e) => {
                        e.stopPropagation();
                        moveImage(index, 'down');
                    });
                    
                    actions.appendChild(moveUpBtn);
                    actions.appendChild(moveDownBtn);
                }
                
                actions.appendChild(deleteBtn);
                
                imageItem.appendChild(img);
                imageItem.appendChild(actions);
                imageList.appendChild(imageItem);
            });
        }
        
        // Remove an image from the list
        function removeImage(index) {
            uploadedImages.splice(index, 1);
            updateImageList();
        }
        
        // Move an image up or down in the list
        function moveImage(index, direction) {
            if (direction === 'up' && index > 0) {
                [uploadedImages[index], uploadedImages[index - 1]] = [uploadedImages[index - 1], uploadedImages[index]];
            } else if (direction === 'down' && index < uploadedImages.length - 1) {
                [uploadedImages[index], uploadedImages[index + 1]] = [uploadedImages[index + 1], uploadedImages[index]];
            }
            updateImageList();
        }
        
        // Clear all images
        function clearAllImages() {
            uploadedImages = [];
            updateImageList();
            hideMessage(errorMessage);
            hideMessage(successMessage);
        }
        
        // Generate PDF from images
        async function generatePdf() {
            if (uploadedImages.length === 0) {
                showMessage(errorMessage, 'No images to convert.');
                return;
            }
            
            // Show loading indicator
            loadingIndicator.style.display = 'block';
            generatePdfBtn.disabled = true;
            hideMessage(errorMessage);
            hideMessage(successMessage);
            
            try {
                // Get PDF settings
                const pageSize = pageSizeSelect.value;
                const orientation = orientationSelect.value;
                const margin = parseInt(marginInput.value);
                const imageFit = imageFitSelect.value;
                
                // Create a new PDF document
                const pdf = new jsPDF({
                    orientation: orientation,
                    unit: 'mm',
                    format: pageSize
                });
                
                // Process each image
                for (let i = 0; i < uploadedImages.length; i++) {
                    const image = uploadedImages[i];
                    
                    // Add a new page for each image after the first one
                    if (i > 0) {
                        pdf.addPage(pageSize, orientation);
                    }
                    
                    // Calculate dimensions to fit the image on the page
                    const pageWidth = pdf.internal.pageSize.getWidth();
                    const pageHeight = pdf.internal.pageSize.getHeight();
                    
                    let imgWidth, imgHeight;
                    
                    if (imageFit === 'contain') {
                        // Fit the whole image within the page while maintaining aspect ratio
                        const ratio = Math.min(
                            (pageWidth - margin * 2) / image.width,
                            (pageHeight - margin * 2) / image.height
                        );
                        imgWidth = image.width * ratio;
                        imgHeight = image.height * ratio;
                    } else {
                        // Cover the page, potentially cropping the image
                        const pageRatio = (pageWidth - margin * 2) / (pageHeight - margin * 2);
                        const imageRatio = image.width / image.height;
                        
                        if (imageRatio > pageRatio) {
                            imgWidth = pageWidth - margin * 2;
                            imgHeight = imgWidth / imageRatio;
                        } else {
                            imgHeight = pageHeight - margin * 2;
                            imgWidth = imgHeight * imageRatio;
                        }
                    }
                    
                    // Center the image on the page
                    const x = (pageWidth - imgWidth) / 2;
                    const y = (pageHeight - imgHeight) / 2;
                    
                    // Add the image to the PDF
                    pdf.addImage(image.url, 'JPEG', x, y, imgWidth, imgHeight);
                }
                
                // Generate the PDF and create a download link
                const pdfBlob = pdf.output('blob');
                const pdfUrl = URL.createObjectURL(pdfBlob);
                
                // Create a temporary link to download the PDF
                const downloadLink = document.createElement('a');
                downloadLink.href = pdfUrl;
                downloadLink.download = 'images.pdf';
                document.body.appendChild(downloadLink);
                downloadLink.click();
                document.body.removeChild(downloadLink);
                
                // Clean up
                URL.revokeObjectURL(pdfUrl);
                
                showMessage(successMessage, 'PDF generated successfully!');
            } catch (error) {
                console.error('Error generating PDF:', error);
                showMessage(errorMessage, 'An error occurred while generating the PDF. Please try again.');
            } finally {
                // Hide loading indicator
                loadingIndicator.style.display = 'none';
                generatePdfBtn.disabled = false;
            }
        }
        
        // Show a message
        function showMessage(element, text) {
            element.textContent = text;
            element.style.display = 'block';
        }
        
        // Hide a message
        function hideMessage(element) {
            element.style.display = 'none';
        }
        
        // Theme management
        function checkThemePreference() {
            const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
            const savedTheme = localStorage.getItem('theme');
            
            if (savedTheme) {
                document.documentElement.setAttribute('data-theme', savedTheme);
            } else if (prefersDark) {
                document.documentElement.setAttribute('data-theme', 'dark');
            }
        }
        
        function toggleTheme() {
            const currentTheme = document.documentElement.getAttribute('data-theme');
            const newTheme = currentTheme === 'dark' ? 'light' : 'dark';
            
            document.documentElement.setAttribute('data-theme', newTheme);
            localStorage.setItem('theme', newTheme);
        }
        
        // Initialize the app when the DOM is loaded
        document.addEventListener('DOMContentLoaded', init);
    </script>
</body>
</html>
