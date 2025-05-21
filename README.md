<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PDF Master Pro | Complete PDF Solutions</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf-lib/1.17.1/pdf-lib.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.11.338/pdf.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Download.js/1.4.7/download.min.js"></script>
    <style>
        /* All the CSS styles from the previous code remain exactly the same */
        /* ... */
    </style>
</head>
<body>
    <!-- All the HTML structure from the previous code remains exactly the same -->
    <!-- ... -->

    <script>
        // Set PDF.js worker path
        pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.11.338/pdf.worker.min.js';

        // Initialize jsPDF
        const { jsPDF } = window.jspdf;

        // DOM Elements
        const pages = {
            home: document.getElementById('home-page'),
            textToPdf: document.getElementById('text-to-pdf-page'),
            imagesToPdf: document.getElementById('images-to-pdf-page'),
            pdfLocker: document.getElementById('pdf-locker-page'),
            pdfViewer: document.getElementById('pdf-viewer-page')
        };

        // Initialize the app
        function init() {
            setupNavigation();
            setupTextToPDF();
            setupImagesToPDF();
            setupPDFLocker();
            setupPDFViewer();
        }

        // Setup page navigation
        function setupNavigation() {
            const navLinks = document.querySelectorAll('[data-page]');
            const mobileMenuBtn = document.getElementById('mobileMenuBtn');
            const navMenu = document.getElementById('navLinks');

            navLinks.forEach(link => {
                link.addEventListener('click', (e) => {
                    e.preventDefault();
                    const page = link.dataset.page;
                    showPage(page);
                    
                    // Update active nav link
                    document.querySelectorAll('.nav-links a').forEach(navLink => {
                        navLink.classList.remove('active');
                        if (navLink.dataset.page === page) {
                            navLink.classList.add('active');
                        }
                    });
                });
            });

            // Mobile menu toggle
            mobileMenuBtn.addEventListener('click', () => {
                navMenu.classList.toggle('active');
            });
        }

        // Show specific page
        function showPage(pageId) {
            // Hide all pages
            Object.values(pages).forEach(page => {
                page.classList.remove('active');
            });

            // Show selected page
            switch(pageId) {
                case 'home':
                    pages.home.classList.add('active');
                    break;
                case 'text-to-pdf':
                    pages.textToPdf.classList.add('active');
                    break;
                case 'images-to-pdf':
                    pages.imagesToPdf.classList.add('active');
                    break;
                case 'pdf-locker':
                    pages.pdfLocker.classList.add('active');
                    break;
                case 'pdf-viewer':
                    pages.pdfViewer.classList.add('active');
                    break;
            }

            // Close mobile menu if open
            document.getElementById('navLinks').classList.remove('active');
        }

        // Text to PDF functionality
        function setupTextToPDF() {
            const createPdfBtn = document.getElementById('create-pdf-btn');
            const createPdfBtnText = document.getElementById('create-pdf-btn-text');
            const progressContainer = document.getElementById('text-pdf-progress');
            const progressBar = document.getElementById('text-pdf-progress-bar');
            const statusMessage = document.getElementById('text-to-pdf-status');

            createPdfBtn.addEventListener('click', async () => {
                const textContent = document.getElementById('text-content').value;
                const pdfFilename = document.getElementById('pdf-filename').value || 'document.pdf';
                const addPageNumbers = document.getElementById('add-page-numbers').checked;
                const addTimestamp = document.getElementById('add-timestamp').checked;

                if (!textContent.trim()) {
                    showStatusMessage(statusMessage, 'Please enter some text to convert', 'error');
                    return;
                }

                try {
                    createPdfBtn.disabled = true;
                    createPdfBtnText.innerHTML = '<span class="spinner"></span> Creating PDF...';
                    progressContainer.style.display = 'block';
                    progressBar.style.width = '10%';

                    // Create new PDF document
                    const doc = new jsPDF();
                    const lines = doc.splitTextToSize(textContent, 180);
                    const pageHeight = doc.internal.pageSize.height;
                    let y = 20;
                    
                    progressBar.style.width = '30%';

                    // Add text content
                    for (let i = 0; i < lines.length; i++) {
                        if (y > pageHeight - 20) {
                            doc.addPage();
                            y = 20;
                            
                            if (addPageNumbers) {
                                const pageCount = doc.internal.getNumberOfPages();
                                doc.setFontSize(10);
                                doc.text(`Page ${pageCount - 1} of ${pageCount}`, 105, pageHeight - 10, { align: 'center' });
                            }
                        }
                        doc.text(lines[i], 15, y);
                        y += 7;
                    }

                    progressBar.style.width = '60%';

                    // Add metadata
                    if (addTimestamp) {
                        doc.setProperties({
                            title: pdfFilename.replace('.pdf', ''),
                            subject: 'Created with PDF Master Pro',
                            author: 'PDF Master Pro User',
                            creator: 'PDF Master Pro',
                            creationDate: new Date()
                        });
                    }

                    progressBar.style.width = '80%';

                    // Add page numbers if enabled
                    if (addPageNumbers) {
                        const pageCount = doc.internal.getNumberOfPages();
                        for (let i = 1; i <= pageCount; i++) {
                            doc.setPage(i);
                            doc.setFontSize(10);
                            doc.text(`Page ${i} of ${pageCount}`, 105, pageHeight - 10, { align: 'center' });
                        }
                    }

                    progressBar.style.width = '100%';

                    // Save the PDF
                    doc.save(pdfFilename);
                    
                    showStatusMessage(statusMessage, 'PDF created successfully!', 'success');
                } catch (error) {
                    console.error('Error creating PDF:', error);
                    showStatusMessage(statusMessage, 'Error creating PDF: ' + error.message, 'error');
                } finally {
                    createPdfBtn.disabled = false;
                    createPdfBtnText.textContent = 'Create PDF';
                    setTimeout(() => {
                        progressContainer.style.display = 'none';
                        progressBar.style.width = '0%';
                    }, 1000);
                }
            });
        }

        // Images to PDF functionality
        function setupImagesToPDF() {
            const imagesUpload = document.getElementById('images-upload');
            const imagesInput = document.getElementById('images-input');
            const imagesInfo = document.getElementById('images-info');
            const createPdfBtn = document.getElementById('create-images-pdf-btn');
            const createPdfBtnText = document.getElementById('create-images-pdf-btn-text');
            const progressContainer = document.getElementById('images-pdf-progress');
            const progressBar = document.getElementById('images-pdf-progress-bar');
            const statusMessage = document.getElementById('images-to-pdf-status');
            let selectedImages = [];

            // Handle file selection
            imagesUpload.addEventListener('click', () => imagesInput.click());
            imagesUpload.addEventListener('dragover', (e) => {
                e.preventDefault();
                imagesUpload.style.borderColor = '#4361ee';
                imagesUpload.style.backgroundColor = 'rgba(67, 97, 238, 0.05)';
            });
            imagesUpload.addEventListener('dragleave', () => {
                imagesUpload.style.borderColor = '#ddd';
                imagesUpload.style.backgroundColor = 'transparent';
            });
            imagesUpload.addEventListener('drop', (e) => {
                e.preventDefault();
                imagesUpload.style.borderColor = '#ddd';
                imagesUpload.style.backgroundColor = 'transparent';
                if (e.dataTransfer.files.length > 0) {
                    handleImageFiles(e.dataTransfer.files);
                }
            });
            imagesInput.addEventListener('change', () => {
                if (imagesInput.files.length > 0) {
                    handleImageFiles(imagesInput.files);
                }
            });

            function handleImageFiles(files) {
                selectedImages = [];
                const imageFiles = Array.from(files).filter(file => file.type.startsWith('image/'));
                
                if (imageFiles.length === 0) {
                    showStatusMessage(statusMessage, 'Please select valid image files', 'error');
                    return;
                }

                // Display file info
                imagesInfo.textContent = `${imageFiles.length} image(s) selected`;
                createPdfBtn.disabled = false;
                
                // Store files for processing
                selectedImages = imageFiles;
            }

            // Create PDF from images
            createPdfBtn.addEventListener('click', async () => {
                if (selectedImages.length === 0) {
                    showStatusMessage(statusMessage, 'Please select at least one image', 'error');
                    return;
                }

                const pdfFilename = document.getElementById('images-pdf-filename').value || 'images.pdf';
                const layout = document.getElementById('image-layout').value;

                try {
                    createPdfBtn.disabled = true;
                    createPdfBtnText.innerHTML = '<span class="spinner"></span> Creating PDF...';
                    progressContainer.style.display = 'block';
                    progressBar.style.width = '5%';

                    const doc = new jsPDF();
                    let position = { x: 10, y: 10 };
                    const pageWidth = doc.internal.pageSize.width;
                    const pageHeight = doc.internal.pageSize.height;
                    let gridCols = 1, gridRows = 1;
                    
                    // Set grid layout
                    if (layout === 'grid-2x2') {
                        gridCols = 2;
                        gridRows = 2;
                    } else if (layout === 'grid-3x3') {
                        gridCols = 3;
                        gridRows = 3;
                    }

                    const cellWidth = (pageWidth - 20) / gridCols;
                    const cellHeight = (pageHeight - 20) / gridRows;
                    let currentCol = 0, currentRow = 0;

                    for (let i = 0; i < selectedImages.length; i++) {
                        const imageFile = selectedImages[i];
                        const imageUrl = URL.createObjectURL(imageFile);
                        
                        await new Promise((resolve) => {
                            const img = new Image();
                            img.onload = async () => {
                                // Calculate dimensions to fit in cell while maintaining aspect ratio
                                let imgWidth = img.width;
                                let imgHeight = img.height;
                                const ratio = Math.min(
                                    (cellWidth - 10) / imgWidth,
                                    (cellHeight - 10) / imgHeight
                                );
                                imgWidth *= ratio;
                                imgHeight *= ratio;
                                
                                // Center image in cell
                                const x = position.x + (cellWidth - imgWidth) / 2;
                                const y = position.y + (cellHeight - imgHeight) / 2;
                                
                                // Add image to PDF
                                doc.addImage(img, 'JPEG', x, y, imgWidth, imgHeight);
                                
                                // Update progress
                                const progress = Math.floor(((i + 1) / selectedImages.length) * 90) + 5;
                                progressBar.style.width = `${progress}%`;
                                
                                // Move to next position
                                currentCol++;
                                if (currentCol >= gridCols) {
                                    currentCol = 0;
                                    currentRow++;
                                    
                                    if (currentRow >= gridRows) {
                                        currentRow = 0;
                                        if (i < selectedImages.length - 1) {
                                            doc.addPage();
                                        }
                                    }
                                }
                                
                                position = {
                                    x: 10 + (currentCol * cellWidth),
                                    y: 10 + (currentRow * cellHeight)
                                };
                                
                                URL.revokeObjectURL(imageUrl);
                                resolve();
                            };
                            img.src = imageUrl;
                        });
                    }

                    progressBar.style.width = '100%';
                    doc.save(pdfFilename);
                    
                    showStatusMessage(statusMessage, 'PDF created successfully!', 'success');
                } catch (error) {
                    console.error('Error creating PDF:', error);
                    showStatusMessage(statusMessage, 'Error creating PDF: ' + error.message, 'error');
                } finally {
                    createPdfBtn.disabled = false;
                    createPdfBtnText.textContent = 'Create PDF';
                    setTimeout(() => {
                        progressContainer.style.display = 'none';
                        progressBar.style.width = '0%';
                    }, 1000);
                }
            });
        }

        // PDF Locker functionality
        function setupPDFLocker() {
            const pdfLockerUpload = document.getElementById('pdf-locker-upload');
            const pdfLockerInput = document.getElementById('pdf-locker-input');
            const pdfLockerInfo = document.getElementById('pdf-locker-info');
            const lockPdfBtn = document.getElementById('lock-pdf-btn');
            const lockPdfBtnText = document.getElementById('lock-pdf-btn-text');
            const progressContainer = document.getElementById('pdf-locker-progress');
            const progressBar = document.getElementById('pdf-locker-progress-bar');
            const statusMessage = document.getElementById('pdf-locker-status');
            let selectedPdf = null;

            // Handle file selection
            pdfLockerUpload.addEventListener('click', () => pdfLockerInput.click());
            pdfLockerUpload.addEventListener('dragover', (e) => {
                e.preventDefault();
                pdfLockerUpload.style.borderColor = '#4361ee';
                pdfLockerUpload.style.backgroundColor = 'rgba(67, 97, 238, 0.05)';
            });
            pdfLockerUpload.addEventListener('dragleave', () => {
                pdfLockerUpload.style.borderColor = '#ddd';
                pdfLockerUpload.style.backgroundColor = 'transparent';
            });
            pdfLockerUpload.addEventListener('drop', (e) => {
                e.preventDefault();
                pdfLockerUpload.style.borderColor = '#ddd';
                pdfLockerUpload.style.backgroundColor = 'transparent';
                if (e.dataTransfer.files.length > 0) {
                    handlePdfFile(e.dataTransfer.files[0]);
                }
            });
            pdfLockerInput.addEventListener('change', () => {
                if (pdfLockerInput.files.length > 0) {
                    handlePdfFile(pdfLockerInput.files[0]);
                }
            });

            function handlePdfFile(file) {
                if (!file.type.includes('pdf')) {
                    showStatusMessage(statusMessage, 'Please select a valid PDF file', 'error');
                    return;
                }

                selectedPdf = file;
                pdfLockerInfo.textContent = file.name;
                lockPdfBtn.disabled = false;
            }

            // Lock PDF with password
            lockPdfBtn.addEventListener('click', async () => {
                if (!selectedPdf) {
                    showStatusMessage(statusMessage, 'Please select a PDF file', 'error');
                    return;
                }

                const password = document.getElementById('lock-password').value;
                const confirmPassword = document.getElementById('confirm-lock-password').value;
                
                if (!password || !confirmPassword) {
                    showStatusMessage(statusMessage, 'Please enter and confirm password', 'error');
                    return;
                }
                
                if (password.length < 6) {
                    showStatusMessage(statusMessage, 'Password must be at least 6 characters', 'error');
                    return;
                }
                
                if (password !== confirmPassword) {
                    showStatusMessage(statusMessage, 'Passwords do not match', 'error');
                    return;
                }

                const allowPrinting = document.getElementById('allow-printing').checked;
                const allowCopying = document.getElementById('allow-copying').checked;
                const allowModification = document.getElementById('allow-modification').checked;

                try {
                    lockPdfBtn.disabled = true;
                    lockPdfBtnText.innerHTML = '<span class="spinner"></span> Locking PDF...';
                    progressContainer.style.display = 'block';
                    progressBar.style.width = '20%';

                    // Read the PDF file
                    const arrayBuffer = await selectedPdf.arrayBuffer();
                    const pdfDoc = await PDFLib.PDFDocument.load(arrayBuffer);
                    
                    progressBar.style.width = '50%';

                    // Set permissions
                    const permissions = {
                        print: allowPrinting,
                        copy: allowCopying,
                        modify: allowModification
                    };

                    // Encrypt the PDF
                    await pdfDoc.encrypt({
                        userPassword: password,
                        ownerPassword: password + '-owner', // Different owner password
                        permissions: permissions
                    });

                    progressBar.style.width = '80%';

                    // Save the encrypted PDF
                    const encryptedPdfBytes = await pdfDoc.save();
                    const blob = new Blob([encryptedPdfBytes], { type: 'application/pdf' });
                    
                    // Generate filename
                    const originalName = selectedPdf.name.replace('.pdf', '');
                    const newFilename = `${originalName}_locked.pdf`;
                    
                    // Download the file
                    download(blob, newFilename, "application/pdf");
                    
                    progressBar.style.width = '100%';
                    showStatusMessage(statusMessage, 'PDF locked successfully!', 'success');
                } catch (error) {
                    console.error('Error locking PDF:', error);
                    showStatusMessage(statusMessage, 'Error locking PDF: ' + error.message, 'error');
                } finally {
                    lockPdfBtn.disabled = false;
                    lockPdfBtnText.textContent = 'Lock PDF';
                    setTimeout(() => {
                        progressContainer.style.display = 'none';
                        progressBar.style.width = '0%';
                    }, 1000);
                }
            });
        }

        // PDF Viewer functionality
        function setupPDFViewer() {
            const pdfViewerUpload = document.getElementById('pdf-viewer-upload');
            const pdfViewerInput = document.getElementById('pdf-viewer-input');
            const pdfViewerInfo = document.getElementById('pdf-viewer-info');
            const statusMessage = document.getElementById('pdf-viewer-status');
            const viewerContainer = document.getElementById('pdf-viewer-container');
            const pdfRender = document.getElementById('pdf-render');
            const pageNum = document.getElementById('page-num');
            const pageCount = document.getElementById('page-count');
            const pageSelect = document.getElementById('page-select');
            const zoomLevel = document.getElementById('zoom-level');
            
            let pdfDoc = null,
                pageNumDisplay = 1,
                pageRendering = false,
                pageNumPending = null,
                scale = 1.0,
                rotation = 0;

            // Handle file selection
            pdfViewerUpload.addEventListener('click', () => pdfViewerInput.click());
            pdfViewerUpload.addEventListener('dragover', (e) => {
                e.preventDefault();
                pdfViewerUpload.style.borderColor = '#4361ee';
                pdfViewerUpload.style.backgroundColor = 'rgba(67, 97, 238, 0.05)';
            });
            pdfViewerUpload.addEventListener('dragleave', () => {
                pdfViewerUpload.style.borderColor = '#ddd';
                pdfViewerUpload.style.backgroundColor = 'transparent';
            });
            pdfViewerUpload.addEventListener('drop', (e) => {
                e.preventDefault();
                pdfViewerUpload.style.borderColor = '#ddd';
                pdfViewerUpload.style.backgroundColor = 'transparent';
                if (e.dataTransfer.files.length > 0) {
                    handlePdfViewerFile(e.dataTransfer.files[0]);
                }
            });
            pdfViewerInput.addEventListener('change', () => {
                if (pdfViewerInput.files.length > 0) {
                    handlePdfViewerFile(pdfViewerInput.files[0]);
                }
            });

            function handlePdfViewerFile(file) {
                if (!file.type.includes('pdf')) {
                    showStatusMessage(statusMessage, 'Please select a valid PDF file', 'error');
                    return;
                }

                pdfViewerInfo.textContent = file.name;
                viewerContainer.style.display = 'block';
                
                // Read the PDF file
                const fileReader = new FileReader();
                fileReader.onload = function() {
                    const typedArray = new Uint8Array(this.result);
                    
                    // Load the PDF document
                    pdfjsLib.getDocument(typedArray).promise.then(function(pdf) {
                        pdfDoc = pdf;
                        pageCount.textContent = pdf.numPages;
                        pageSelect.max = pdf.numPages;
                        
                        // Reset viewer state
                        pageNumDisplay = 1;
                        scale = 1.0;
                        rotation = 0;
                        zoomLevel.textContent = '100%';
                        
                        // Render the first page
                        renderPage(pageNumDisplay);
                    }).catch(function(error) {
                        console.error('Error loading PDF:', error);
                        showStatusMessage(statusMessage, 'Error loading PDF: ' + error.message, 'error');
                    });
                };
                fileReader.readAsArrayBuffer(file);
            }

            // Render a page
            function renderPage(num) {
                pageRendering = true;
                
                pdfDoc.getPage(num).then(function(page) {
                    const viewport = page.getViewport({ scale: scale, rotation: rotation });
                    pdfRender.innerHTML = '';
                    
                    // Set canvas dimensions
                    const canvas = document.createElement('canvas');
                    const context = canvas.getContext('2d');
                    canvas.height = viewport.height;
                    canvas.width = viewport.width;
                    pdfRender.appendChild(canvas);
                    
                    // Render PDF page
                    const renderContext = {
                        canvasContext: context,
                        viewport: viewport
                    };
                    
                    const renderTask = page.render(renderContext);
                    
                    renderTask.promise.then(function() {
                        pageRendering = false;
                        if (pageNumPending !== null) {
                            renderPage(pageNumPending);
                            pageNumPending = null;
                        }
                    });
                    
                    // Update page display
                    pageNum.textContent = num;
                    pageSelect.value = num;
                });
            }

            // Queue a page rendering if another is in progress
            function queueRenderPage(num) {
                if (pageRendering) {
                    pageNumPending = num;
                } else {
                    renderPage(num);
                }
            }

            // Previous page button
            document.getElementById('prev-page').addEventListener('click', function() {
                if (pageNumDisplay <= 1 || !pdfDoc) return;
                pageNumDisplay--;
                queueRenderPage(pageNumDisplay);
            });

            // Next page button
            document.getElementById('next-page').addEventListener('click', function() {
                if (pageNumDisplay >= pdfDoc.numPages || !pdfDoc) return;
                pageNumDisplay++;
                queueRenderPage(pageNumDisplay);
            });

            // Go to page input
            document.getElementById('go-to-page').addEventListener('click', function() {
                if (!pdfDoc) return;
                const num = parseInt(pageSelect.value);
                if (num >= 1 && num <= pdfDoc.numPages) {
                    pageNumDisplay = num;
                    queueRenderPage(pageNumDisplay);
                }
            });

            // Zoom in button
            document.getElementById('zoom-in').addEventListener('click', function() {
                if (!pdfDoc) return;
                scale *= 1.2;
                zoomLevel.textContent = Math.round(scale * 100) + '%';
                queueRenderPage(pageNumDisplay);
            });

            // Zoom out button
            document.getElementById('zoom-out').addEventListener('click', function() {
                if (!pdfDoc) return;
                scale /= 1.2;
                zoomLevel.textContent = Math.round(scale * 100) + '%';
                queueRenderPage(pageNumDisplay);
            });

            // Rotate left button
            document.getElementById('rotate-left').addEventListener('click', function() {
                if (!pdfDoc) return;
                rotation = (rotation - 90) % 360;
                queueRenderPage(pageNumDisplay);
            });

            // Rotate right button
            document.getElementById('rotate-right').addEventListener('click', function() {
                if (!pdfDoc) return;
                rotation = (rotation + 90) % 360;
                queueRenderPage(pageNumDisplay);
            });
        }

        // Helper function to show status messages
        function showStatusMessage(element, message, type) {
            element.textContent = message;
            element.className = 'status-message ' + type;
            setTimeout(() => {
                element.className = 'status-message';
                element.textContent = '';
            }, 5000);
        }

        // Initialize the app when DOM is loaded
        document.addEventListener('DOMContentLoaded', init);
    </script>
</body>
</html>
