<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="AI-powered focus timer that detects when you're studying using DeepSeek AI. Improve productivity with real-time attention tracking.">
    <meta name="keywords" content="focus timer, study timer, AI productivity, DeepSeek AI, concentration tracker, Pomodoro alternative">
    <meta name="author" content="FocusStudyPro">
    <meta property="og:title" content="AI Focus Timer | DeepSeek-Powered Study Assistant">
    <meta property="og:description" content="Track study sessions with AI-powered focus detection. Uses DeepSeek AI for real-time attention analysis.">
    <meta property="og:type" content="website">
    <meta property="og:url" content="https://focusstudypro.com">
    <meta property="og:image" content="https://focusstudypro.com/preview.jpg">
    <meta name="twitter:card" content="summary_large_image">
    <title>AI Focus Timer | DeepSeek-Powered Study Assistant</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google AdSense -->
    <script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-YOUR_ADSENSE_ID" crossorigin="anonymous"></script>
    <style>
        :root {
            --primary: #1e40af;
            --secondary: #1e3a8a;
            --success: #10b981;
            --warning: #f59e0b;
            --danger: #ef4444;
        }
        body {
            font-family: 'Inter', sans-serif;
            background: linear-gradient(to bottom, #1e3a8a, #111827);
            min-height: 100vh;
            color: white;
        }
        .flip-digit {
            perspective: 1000px;
        }
        .flip-digit-inner {
            position: relative;
            width: 100%;
            height: 100%;
            transition: transform 0.6s;
            transform-style: preserve-3d;
        }
        .flip-digit.flipped .flip-digit-inner {
            transform: rotateX(180deg);
        }
        .flip-digit-front, .flip-digit-back {
            position: absolute;
            width: 100%;
            height: 100%;
            backface-visibility: hidden;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .flip-digit-back {
            transform: rotateX(180deg);
        }
        #video {
            transform: scaleX(-1);
        }
        #canvas {
            transform: scaleX(-1);
        }
        .ad-container {
            background: rgba(255, 255, 255, 0.1);
            border-radius: 12px;
            padding: 16px;
            text-align: center;
        }
        .permission-overlay {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(0,0,0,0.9);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 1000;
            padding: 20px;
            text-align: center;
        }
        .permission-content {
            max-width: 500px;
            background: #1e293b;
            padding: 30px;
            border-radius: 12px;
        }
        #manualControls {
            display: none;
        }
    </style>
</head>
<body>
    <!-- Camera Permission Overlay -->
    <div id="permissionOverlay" class="permission-overlay">
        <div class="permission-content">
            <i class="fas fa-camera text-4xl text-blue-400 mb-4"></i>
            <h2 class="text-2xl font-bold mb-4">Camera Access Required</h2>
            <p class="mb-6 text-gray-300">To detect your focus level accurately, we need access to your camera. The AI will analyze your head position and eye contact to track your study focus in real-time.</p>
            <div class="flex flex-col sm:flex-row gap-4">
                <button id="denyCameraBtn" class="px-6 py-3 bg-gray-700 hover:bg-gray-600 rounded-lg transition">
                    Continue Without Camera
                </button>
                <button id="allowCameraBtn" class="px-6 py-3 bg-blue-600 hover:bg-blue-500 rounded-lg transition">
                    <i class="fas fa-camera mr-2"></i> Allow Camera Access
                </button>
            </div>
            <p class="mt-4 text-sm text-gray-400">All processing happens locally in your browser. No video data is stored or sent to servers.</p>
        </div>
    </div>

    <!-- Manual Controls (hidden by default) -->
    <div id="manualControls" class="flex justify-center gap-4 mb-6">
        <button id="startBtn" class="px-6 py-3 bg-green-600 hover:bg-green-500 rounded-lg transition">
            <i class="fas fa-play mr-2"></i> Start Studying
        </button>
        <button id="pauseBtn" class="px-6 py-3 bg-yellow-600 hover:bg-yellow-500 rounded-lg transition">
            <i class="fas fa-pause mr-2"></i> Take Break
        </button>
    </div>

    <!-- Rest of your existing HTML remains here -->
    <header class="container mx-auto px-4 py-6 flex justify-between items-center">
        <!-- ... existing header content ... -->
    </header>

    <main class="container mx-auto px-4">
        <!-- ... all your existing main content ... -->
    </main>

    <footer class="bg-gray-900 py-12">
        <!-- ... existing footer content ... -->
    </footer>

    <!-- Settings Modal -->
    <div id="settingsModal" class="fixed inset-0 bg-black bg-opacity-70 flex items-center justify-center z-50 hidden">
        <!-- ... existing modal content ... -->
    </div>

    <script>
        // State management
        const state = {
            timer: 0,
            focusedTime: 0,
            distractedTime: 0,
            awayTime: 0,
            isRunning: false,
            status: 'away', // 'studying', 'distracted', 'away'
            lastStatusChange: Date.now(),
            detectionInterval: null,
            timerInterval: null,
            settings: {
                sensitivity: 2, // 1=low, 2=medium, 3=high
                distractionAlerts: true,
                focusRegainedAlerts: true
            },
            detectionHistory: [],
            videoStream: null,
            usingCamera: false
        };

        // DOM Elements
        const elements = {
            permissionOverlay: document.getElementById('permissionOverlay'),
            allowCameraBtn: document.getElementById('allowCameraBtn'),
            denyCameraBtn: document.getElementById('denyCameraBtn'),
            manualControls: document.getElementById('manualControls'),
            startBtn: document.getElementById('startBtn'),
            pauseBtn: document.getElementById('pauseBtn'),
            // ... all your existing element references ...
        };

        // Initialize the app
        document.addEventListener('DOMContentLoaded', async () => {
            // Request notification permission
            if ('Notification' in window) {
                Notification.requestPermission();
            }
            
            // Initialize AdSense
            (adsbygoogle = window.adsbygoogle || []).push({});
            
            // Check for existing camera permissions
            try {
                const devices = await navigator.mediaDevices.enumerateDevices();
                const hasCameraPermission = devices.some(device => device.kind === 'videoinput' && device.label);
                
                if (hasCameraPermission) {
                    elements.permissionOverlay.style.display = 'none';
                    initializeAppWithCamera();
                    return;
                }
            } catch (e) {
                console.log("Permission check failed", e);
            }
            
            // Set up permission buttons
            elements.allowCameraBtn.addEventListener('click', async () => {
                try {
                    await initializeAppWithCamera();
                    elements.permissionOverlay.style.display = 'none';
                } catch (error) {
                    alert("Could not access camera. Please check your permissions.");
                    console.error("Camera error:", error);
                    initializeAppWithoutCamera();
                }
            });
            
            elements.denyCameraBtn.addEventListener('click', () => {
                elements.permissionOverlay.style.display = 'none';
                initializeAppWithoutCamera();
                updateStatus("Manual mode - Click to start timer", "yellow");
            });
            
            // Set up manual controls
            elements.startBtn.addEventListener('click', () => {
                setStatus('studying');
            });
            
            elements.pauseBtn.addEventListener('click', () => {
                setStatus('distracted');
            });
            
            // ... rest of your existing event listeners ...
        });
        
        async function initializeAppWithCamera() {
            state.usingCamera = true;
            
            // Request camera access
            try {
                state.videoStream = await navigator.mediaDevices.getUserMedia({
                    video: { 
                        width: { ideal: 640 },
                        height: { ideal: 480 },
                        facingMode: 'user'
                    }
                });
                
                elements.video.srcObject = state.videoStream;
                await new Promise(resolve => {
                    elements.video.onloadedmetadata = resolve;
                });
                elements.video.play();
                
                // Initialize AI detection
                updateStatus("Initializing DeepSeek AI...", "gray");
                await initDeepSeekDetection();
                
                // Start detection
                state.detectionInterval = setInterval(detectFocusState, 1000);
                updateStatus("Ready - Focus detection active", "green");
                
            } catch (error) {
                console.error("Camera initialization failed:", error);
                throw error;
            }
        }
        
        function initializeAppWithoutCamera() {
            state.usingCamera = false;
            elements.manualControls.style.display = 'flex';
            elements.video.style.display = 'none';
            updateStatus("Manual mode - Click to start timer", "yellow");
        }

        // ... rest of your existing functions remain the same ...
        // (initDeepSeekDetection, detectFocusState, setStatus, updateStatusUI, etc.)

        // Clean up camera stream when leaving
        window.addEventListener('beforeunload', () => {
            if (state.videoStream) {
                state.videoStream.getTracks().forEach(track => track.stop());
            }
            // ... rest of your cleanup code ...
        });
    </script>
</body>
</html>
