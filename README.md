<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ai9</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* General styles and font */
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            line-height: 1.6;
            transition: background-color 0.3s, color 0.3s;
        }

        /* Message container and scrollbar customization */
        .message-container {
            max-height: calc(100vh - 120px);
            overflow-y: auto;
            -ms-overflow-style: none;
            scrollbar-width: none;
        }
        .message-container::-webkit-scrollbar {
            display: none;
        }

        /* Individual message box styling */
        .message-box {
            max-width: 80%;
            border-radius: 1.25rem;
            padding: 0.75rem 1rem;
            word-wrap: break-word;
            box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
        }
        .sent-message {
            background-color: #10a37f;
            color: white;
            align-self: flex-end;
        }
        .received-message {
            background-color: #e5e5e5;
            color: #374151;
            align-self: flex-start;
        }

        /* Modal styling */
        .modal-overlay {
            background-color: rgba(0, 0, 0, 0.5);
        }
        .modal-content {
            max-height: 80vh;
            overflow-y: auto;
            -ms-overflow-style: none;
            scrollbar-width: none;
        }
        .modal-content::-webkit-scrollbar {
            display: none;
        }

        /* Dark mode styling */
        .dark-mode {
            background-color: #343541;
            color: #ececf1;
        }
        .dark-mode .received-message {
            background-color: #40414f;
            color: #d1d5db;
        }
        .dark-mode .header, .dark-mode .input-area {
            background-color: #343541;
        }
        .dark-mode #message-input {
            background-color: #40414f;
            color: #ececf1;
            border-color: #555666;
        }
        .dark-mode #send-button, .dark-mode #mic-button {
            background-color: #10a37f;
        }
        .dark-mode #settings-button {
            background-color: #40414f;
        }
        .dark-mode .modal-content {
            background-color: #343541;
        }

        /* Mic pulse animation */
        .mic-active {
            animation: pulse 1s infinite;
        }
        @keyframes pulse {
            0% { box-shadow: 0 0 0 0 rgba(16, 163, 127, 0.4); }
            70% { box-shadow: 0 0 0 10px rgba(16, 163, 127, 0); }
            100% { box-shadow: 0 0 0 0 rgba(16, 163, 127, 0); }
        }
    </style>
</head>
<body class="bg-gray-100 dark:bg-gray-900 flex flex-col h-screen">
    <div class="container mx-auto max-w-3xl flex flex-col h-full bg-white dark:bg-gray-800 rounded-lg shadow-lg">

        <!-- Header -->
        <div class="header bg-white dark:bg-gray-800 text-gray-800 dark:text-white p-4 shadow-md flex items-center justify-between sticky top-0 z-10 rounded-t-lg">
            <h1 class="text-xl font-bold">ai9</h1>
            <div class="flex items-center space-x-2">
                <!-- Settings button on the far left -->
                <button id="settings-button" class="text-gray-600 dark:text-gray-300 focus:outline-none p-2 rounded-full hover:bg-gray-200 dark:hover:bg-gray-700 transition-colors duration-200">
                    <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.942 3.313.841 2.37 2.373a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.942 1.543-.841 3.313-2.373 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.942-3.313-.841-2.37-2.373a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.942-1.543.841-3.313 2.373-2.37a1.724 1.724 0 002.572-1.065z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"></path></svg>
                </button>
            </div>
        </div>

        <!-- Messages Area -->
        <div id="messages" class="flex-1 p-4 overflow-y-auto space-y-4 message-container">
            <!-- Initial welcome message -->
            <div class="flex justify-start">
                <div class="message-box received-message">
                    مرحباً بك! أنا هنا لمساعدتك في أي شيء تحتاجه. كيف يمكنني أن أخدمك اليوم؟
                </div>
            </div>
        </div>

        <!-- Input Area -->
        <div class="input-area p-4 bg-gray-100 dark:bg-gray-800 border-t border-gray-200 dark:border-gray-700 rounded-b-lg">
            <div class="flex items-center space-x-2">
                <input type="file" id="file-upload" accept="image/*" class="hidden">
                <button id="file-button" class="bg-gray-300 hover:bg-gray-400 text-gray-800 p-3 rounded-full font-bold transition-colors duration-200 shadow-lg transform active:scale-95 dark:bg-gray-600 dark:text-white dark:hover:bg-gray-500">
                    <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.172 7l-6.586 6.586a2 2 0 102.828 2.828l6.414-6.586a4 4 0 00-5.656-5.656l-6.415 6.585a6 6 0 108.486 8.486l6.586-6.415a2 2 0 10-2.828-2.828l-6.586 6.414a4 4 0 11-5.657-5.657l6.414-6.586a2 2 0 10-2.828-2.828z"></path>
                    </svg>
                </button>
                <input id="message-input" type="text" placeholder="أرسل رسالة..." class="flex-1 py-3 px-4 rounded-full border border-gray-300 dark:border-gray-600 dark:bg-gray-700 dark:text-white focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200 shadow-sm">
                <button id="mic-button" class="bg-blue-500 hover:bg-blue-600 text-white p-3 rounded-full font-bold transition-colors duration-200 shadow-lg transform active:scale-95">
                    <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 1A6 6 0 006 7v6a6 6 0 006 6 6 6 0 006-6V7a6 6 0 00-6-6z"/>
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 13a8 8 0 008 8 8 8 0 008-8h-2a6 6 0 01-12 0H4z"/>
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 19V23M9 23h6"/>
                    </svg>
                </button>
                <button id="send-button" class="bg-blue-500 hover:bg-blue-600 text-white py-3 px-6 rounded-full font-bold transition-colors duration-200 shadow-lg transform active:scale-95">
                    إرسال
                </button>
            </div>
        </div>
    </div>

    <!-- Settings Modal -->
    <div id="settings-modal" class="fixed inset-0 z-50 flex items-center justify-center hidden p-4">
        <div class="absolute inset-0 modal-overlay" id="modal-backdrop"></div>
        <div class="bg-white dark:bg-gray-800 rounded-lg p-6 shadow-xl relative max-w-sm w-full mx-4 sm:max-w-md md:max-w-lg lg:max-w-xl modal-content">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-2xl font-bold text-gray-800 dark:text-white">الإعدادات</h2>
                <button id="close-modal-button" class="text-gray-400 hover:text-gray-600 dark:hover:text-gray-200 transition-colors">
                    <svg class="h-6 w-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                </button>
            </div>
            
            <div class="space-y-6">
                <!-- Theme Section -->
                <div>
                    <h3 class="text-lg font-semibold text-gray-800 dark:text-white mb-2">المظهر</h3>
                    <div class="p-4 bg-gray-100 dark:bg-gray-700 rounded-lg border border-gray-200 dark:border-gray-600">
                        <label for="theme-toggle" class="flex items-center justify-between cursor-pointer">
                            <span class="text-gray-700 dark:text-gray-300">الوضع الليلي</span>
                            <div class="relative">
                                <input type="checkbox" id="theme-toggle" class="sr-only">
                                <div class="block bg-gray-600 w-14 h-8 rounded-full"></div>
                                <div class="dot absolute left-1 top-1 bg-white w-6 h-6 rounded-full transition-transform"></div>
                            </div>
                        </label>
                    </div>
                </div>

                <!-- General Section -->
                <div>
                    <h3 class="text-lg font-semibold text-gray-800 dark:text-white mb-2">عام</h3>
                    <button id="clear-chat-button" class="w-full bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded-full transition-colors duration-200 shadow-md">
                        مسح سجل الدردشة
                    </button>
                </div>
                
                <!-- AI Models Section -->
                <div>
                    <h3 class="text-lg font-semibold text-gray-800 dark:text-white mb-2">نماذج الذكاء الاصطناعي</h3>
                    <select id="model-select" class="w-full py-2 px-3 rounded-md border border-gray-300 dark:border-gray-600 dark:bg-gray-700 dark:text-white focus:outline-none">
                        <option value="gemini-flash">Gemini Flash</option>
                    </select>
                </div>

                <!-- AI Persona Section -->
                <div>
                    <h3 class="text-lg font-semibold text-gray-800 dark:text-white mb-2">شخصية المساعد</h3>
                    <select id="persona-select" class="w-full py-2 px-3 rounded-md border border-gray-300 dark:border-gray-600 dark:bg-gray-700 dark:text-white focus:outline-none">
                        <option value="friendly">صديق ودود</option>
                        <option value="professional">خبير محترف</option>
                        <option value="poet">شاعر</option>
                        <option value="teacher">معلم</option>
                        <option value="philosopher">فيلسوف</option>
                    </select>
                </div>

                <!-- Advanced Settings Section -->
                <div>
                    <h3 class="text-lg font-semibold text-gray-800 dark:text-white mb-2">خصائص متقدمة</h3>
                    <ul class="space-y-3">
                        <li class="flex justify-between items-center text-sm text-gray-700 dark:text-gray-300">
                            <span>تفعيل الذاكرة</span>
                            <input type="checkbox" class="h-4 w-4 text-blue-600 rounded">
                        </li>
                        <li class="flex justify-between items-center text-sm text-gray-700 dark:text-gray-300">
                            <span>درجة الإبداع</span>
                            <input type="range" min="0" max="1" step="0.1" value="0.7" class="h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer">
                        </li>
                    </ul>
                </div>
                
                <!-- Data Management Section -->
                <div>
                    <h3 class="text-lg font-semibold text-gray-800 dark:text-white mb-2">إدارة البيانات</h3>
                    <ul class="space-y-3">
                        <li class="flex justify-between items-center text-sm text-gray-700 dark:text-gray-300">
                            <span>تصدير سجل الدردشة</span>
                            <button class="bg-gray-200 dark:bg-gray-600 text-gray-700 dark:text-gray-300 py-1 px-3 rounded-full text-xs">تصدير</button>
                        </li>
                    </ul>
                </div>
                
                <!-- Help Section -->
                <div>
                    <h3 class="text-lg font-semibold text-gray-800 dark:text-white mb-2">المساعدة</h3>
                    <ul class="space-y-3">
                        <li class="flex justify-between items-center text-sm text-gray-700 dark:text-gray-300">
                            <span>إصدار التطبيق</span>
                            <span>1.2.0</span>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
    
    <!-- Application Logic -->
    <script type="module">
        // Firebase SDKs and App Logic
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, onSnapshot, query, orderBy, serverTimestamp, setDoc, doc, getDoc, getDocs, deleteDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        
        // Global variables required for Firebase/API calls
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
        
        const messageInput = document.getElementById('message-input');
        const sendButton = document.getElementById('send-button');
        const micButton = document.getElementById('mic-button');
        const fileButton = document.getElementById('file-button');
        const fileInput = document.getElementById('file-upload');
        const messagesContainer = document.getElementById('messages');
        const settingsButton = document.getElementById('settings-button');
        const settingsModal = document.getElementById('settings-modal');
        const closeModalButton = document.getElementById('close-modal-button');
        const modalBackdrop = document.getElementById('modal-backdrop');
        const themeToggle = document.getElementById('theme-toggle');
        const clearChatButton = document.getElementById('clear-chat-button');
        const body = document.body;
        const modelSelect = document.getElementById('model-select');

        // Firebase variables
        let db, auth, userId;
        let isAuthReady = false;
        let isGenerating = false;
        let initialMessageRendered = false;

        // --- Firebase Initialization and Auth ---
        function initializeFirebase() {
            try {
                if (Object.keys(firebaseConfig).length === 0) {
                    console.error("Firebase config is missing or empty.");
                    displayMessage('حدث خطأ في تهيئة قاعدة البيانات: الإعدادات مفقودة.', 'received-message');
                    return;
                }
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
                
                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        userId = user.uid;
                        isAuthReady = true;
                        loadChatHistory();
                    } else {
                        // Sign in anonymously if no token is available
                        await signInAnonymously(auth);
                    }
                });
            } catch (e) {
                console.error("Error initializing Firebase:", e);
                displayMessage('حدث خطأ في تهيئة قاعدة البيانات.', 'received-message');
            }
        }
        
        initializeFirebase();

        // --- Chat History Management (Firestore) ---
        function loadChatHistory() {
            if (!isAuthReady) return;

            const chatCollection = collection(db, `artifacts/${appId}/users/${userId}/chats`);
            const q = query(chatCollection, orderBy('timestamp'));

            onSnapshot(q, (snapshot) => {
                // Clear and re-render all messages to ensure correct order
                messagesContainer.innerHTML = '';
                
                snapshot.docs.forEach(doc => {
                    const data = doc.data();
                    if (data.role === 'user') {
                        if (data.text) {
                            displayMessage(data.text, 'sent-message');
                        }
                        if (data.image) {
                            displayImage(data.image.data, 'sent-message');
                        }
                    } else if (data.role === 'model') {
                        if (data.text) {
                            displayMessage(data.text, 'received-message');
                        }
                        if (data.image) {
                            displayImage(data.image.data, 'received-message');
                        }
                    }
                });
            }, (error) => {
                console.error("Error listening to messages:", error);
                displayMessage('فشل تحميل سجل المحادثات.', 'received-message');
            });
        }

        async function saveMessage(role, content) {
            if (!isAuthReady) {
                console.warn("Firebase not ready, cannot save message.");
                return;
            }
            try {
                const chatCollection = collection(db, `artifacts/${appId}/users/${userId}/chats`);
                await addDoc(chatCollection, {
                    role: role,
                    ...content,
                    timestamp: serverTimestamp()
                });
            } catch (e) {
                console.error("Error adding document:", e);
            }
        }

        // --- Speech Recognition API ---
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        const recognitionSupported = !!SpeechRecognition;
        let recognition = null;
        
        if (recognitionSupported) {
            recognition = new SpeechRecognition();
            recognition.continuous = false;
            recognition.lang = 'ar-AR';
            
            recognition.onresult = (event) => {
                const transcript = event.results[0][0].transcript;
                messageInput.value = transcript;
                sendMessage();
            };
            
            recognition.onstart = () => {
                micButton.classList.add('mic-active');
            };
            
            recognition.onend = () => {
                micButton.classList.remove('mic-active');
            };
            
            recognition.onerror = (event) => {
                console.error("Speech recognition error:", event.error);
                if (event.error === 'not-allowed') {
                    displayMessage('عذراً، تحتاج إلى منح الإذن لاستخدام الميكروفون.', 'received-message');
                } else {
                    displayMessage('عذراً، حدث خطأ في التعرف على الصوت. الرجاء المحاولة مرة أخرى.', 'received-message');
                }
            };
            
            micButton.addEventListener('click', () => {
                try {
                    recognition.start();
                } catch (e) {
                    console.error("Error starting speech recognition:", e);
                }
            });
        } else {
            micButton.style.display = 'none';
            console.warn("Speech recognition not supported in this browser.");
        }

        // --- UI Event Listeners ---
        settingsButton.addEventListener('click', () => {
            settingsModal.classList.remove('hidden');
        });

        modalBackdrop.addEventListener('click', () => {
            settingsModal.classList.add('hidden');
        });

        closeModalButton.addEventListener('click', () => {
            settingsModal.classList.add('hidden');
        });

        sendButton.addEventListener('click', sendMessage);
        messageInput.addEventListener('keydown', (event) => {
            if (event.key === 'Enter') {
                sendMessage();
            }
        });
        
        fileButton.addEventListener('click', () => {
            fileInput.click();
        });

        fileInput.addEventListener('change', async (event) => {
            const file = event.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = async (e) => {
                const base64Image = e.target.result;
                saveMessage('user', { image: { data: base64Image, mimeType: file.type } });
                await sendToGemini([{ inlineData: { mimeType: file.type, data: base64Image.split(',')[1] } }]);
                messageInput.focus();
            };
            reader.readAsDataURL(file);
            fileInput.value = ''; // Reset file input
        });

        // --- Keyboard Layout Detection ---
        const arabicRegex = /[\u0600-\u06FF]/;
        const englishRegex = /[a-zA-Z]/;
        let isFirstKey = true;

        messageInput.addEventListener('keydown', (event) => {
            if (isFirstKey) {
                const key = event.key;
                if (englishRegex.test(key)) {
                    messageInput.placeholder = "Send a message...";
                    messageInput.dir = "ltr";
                } else if (arabicRegex.test(key)) {
                    messageInput.placeholder = "أرسل رسالة...";
                    messageInput.dir = "rtl";
                }
                isFirstKey = false;
            }
        });

        messageInput.addEventListener('blur', () => {
            isFirstKey = true;
            messageInput.placeholder = "أرسل رسالة...";
            messageInput.dir = "rtl";
        });
        
        // --- Core Logic (API calls to Gemini) ---
        async function sendToGemini(parts) {
            if (isGenerating) return;
            isGenerating = true;
            messageInput.disabled = true;
            sendButton.disabled = true;

            const tempMessageDiv = displayMessage('جاري الكتابة...', 'received-message', 'temp-loading');

            const payload = {
                contents: [{
                    parts: parts
                }],
                // Enable Google Search grounding for up-to-date and accurate information.
                tools: [{
                    "google_search": {}
                }],
                systemInstruction: {
                    parts: [{
                        // Updated the system instruction to be in Arabic to match the user's language.
                        text: "تصرف كمساعد مفيد، وموجز، وودود. قدم إجابات قصيرة ومباشرة."
                    }]
                }
            };
            
            // This is the fix. We are now using the model that does not require an API key.
            const selectedModel = 'gemini-2.5-flash-preview-05-20';
            const apiKey = "";
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/${selectedModel}:generateContent?key=${apiKey}`;

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                const result = await response.json();
                const text = result?.candidates?.[0]?.content?.parts?.[0]?.text || "عذراً، لم أتمكن من توليد رد.";
                
                tempMessageDiv.remove();
                saveMessage('model', { text: text });
            } catch (e) {
                console.error("API call failed:", e);
                tempMessageDiv.remove();
                displayMessage('عذراً، حدث خطأ. الرجاء المحاولة مرة أخرى.', 'received-message');
            } finally {
                isGenerating = false;
                messageInput.disabled = false;
                sendButton.disabled = false;
                // Focus the input field after the operation completes
                messageInput.focus();
            }
        }

        async function sendMessage() {
            const messageText = messageInput.value.trim();
            if (messageText === '') return;
            
            // Check for the specific questions about the creator
            const creatorQuestions = ["من الذي صنعك", "مين سواك", "من انشأك", "من صممك", "من برمجك", "من قام بتدريبك", "من هو المطور", "من هو المهندس"];
            const isCreatorQuestion = creatorQuestions.some(q => messageText.includes(q));

            if (isCreatorQuestion) {
                const creatorResponse = "أنا من تصميم وبرمجة وتدريب المهندس معتصم بالله.";
                
                // Save messages and display hard-coded response without API call
                saveMessage('user', { text: messageText });
                saveMessage('model', { text: creatorResponse });
                
                displayMessage(creatorResponse, 'received-message');

                messageInput.value = '';
                messageInput.focus();
                return;
            }
            
            saveMessage('user', { text: messageText });
            messageInput.value = '';
            
            await sendToGemini([{ text: messageText }]);
        }
        
        function displayMessage(text, className, id = '') {
            const messageDiv = document.createElement('div');
            messageDiv.className = `flex ${className === 'sent-message' ? 'justify-end' : 'justify-start'}`;
            if (id) messageDiv.id = id;
            
            const messageBox = document.createElement('div');
            messageBox.className = `message-box ${className}`;
            messageBox.textContent = text;
            messageDiv.appendChild(messageBox);
            
            messagesContainer.appendChild(messageDiv);
            messagesContainer.scrollTop = messagesContainer.scrollHeight;
            return messageDiv;
        }

        function displayImage(imageUrl, className) {
            const messageDiv = document.createElement('div');
            messageDiv.className = `flex ${className === 'sent-message' ? 'justify-end' : 'justify-start'}`;
            
            const messageBox = document.createElement('div');
            messageBox.className = `message-box ${className} p-0`;
            
            const imgElement = document.createElement('img');
            imgElement.src = imageUrl;
            imgElement.className = 'w-full h-auto rounded-xl';
            
            messageBox.appendChild(imgElement);
            messageDiv.appendChild(messageBox);
            messagesContainer.appendChild(messageDiv);
            messagesContainer.scrollTop = messagesContainer.scrollHeight;
        }
        
        // Theme toggle functionality
        themeToggle.addEventListener('change', () => {
            if (themeToggle.checked) {
                body.classList.add('dark-mode');
            } else {
                body.classList.remove('dark-mode');
            }
        });

        // Clear chat functionality
        clearChatButton.addEventListener('click', () => {
            const modalMessage = "هل أنت متأكد من أنك تريد مسح جميع المحادثات؟ لا يمكن التراجع عن هذا الإجراء.";
            
            const mockModal = document.createElement('div');
            mockModal.className = 'fixed inset-0 z-50 flex items-center justify-center bg-gray-800 bg-opacity-75';
            mockModal.innerHTML = `
                <div class="bg-white dark:bg-gray-700 rounded-lg p-6 shadow-xl max-w-sm mx-auto">
                    <p class="text-lg font-semibold text-gray-800 dark:text-white mb-4">${modalMessage}</p>
                    <div class="flex justify-end space-x-4">
                        <button id="cancel-clear" class="px-4 py-2 text-gray-600 dark:text-gray-300 rounded-full hover:bg-gray-200 dark:hover:bg-gray-600 transition-colors">إلغاء</button>
                        <button id="confirm-clear" class="px-4 py-2 bg-red-500 text-white rounded-full hover:bg-red-600 transition-colors">تأكيد</button>
                    </div>
                </div>
            `;
            document.body.appendChild(mockModal);
            
            document.getElementById('cancel-clear').addEventListener('click', () => mockModal.remove());
            document.getElementById('confirm-clear').addEventListener('click', async () => {
                if (!isAuthReady) return;
                try {
                    const chatCollection = collection(db, `artifacts/${appId}/users/${userId}/chats`);
                    const q = query(chatCollection);
                    const snapshot = await getDocs(q);
                    const deletePromises = snapshot.docs.map(doc => deleteDoc(doc.ref));
                    await Promise.all(deletePromises);
                    messagesContainer.innerHTML = '';
                    // Re-add the initial welcome message after clearing
                    const welcomeMessageDiv = document.createElement('div');
                    welcomeMessageDiv.className = 'flex justify-start';
                    welcomeMessageDiv.innerHTML = `
                        <div class="message-box received-message">
                            مرحباً بك! أنا هنا لمساعدتك في أي شيء تحتاجه. كيف يمكنني أن أخدمك اليوم؟
                        </div>
                    `;
                    messagesContainer.appendChild(welcomeMessageDiv);
                } catch (e) {
                    console.error("Error clearing chat:", e);
                    displayMessage('فشل في مسح سجل المحادثات.', 'received-message');
                }
                settingsModal.classList.add('hidden');
                mockModal.remove();
            });
        });

        // Initial welcome message check
        if (messagesContainer.children.length === 0) {
            const welcomeMessageDiv = document.createElement('div');
            welcomeMessageDiv.className = 'flex justify-start';
            welcomeMessageDiv.innerHTML = `
                <div class="message-box received-message">
                    مرحباً بك! أنا هنا لمساعدتك في أي شيء تحتاجه. كيف يمكنني أن أخدمك اليوم؟
                </div>
            `;
            messagesContainer.appendChild(welcomeMessageDiv);
        }
    </script>
</body>
</html>
