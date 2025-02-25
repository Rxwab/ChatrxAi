<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Chatbot</title>
    <style>
        /* General Styles */
        body {
            font-family: Arial, sans-serif;
            background-color: #121212;
            color: white;
            margin: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .header {
            width: 100%;
            background-color: #1e1e1e;
            padding: 10px 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .header h1 {
            margin: 0;
        }
        .chat-container {
            width: 100%;
            max-width: 600px;
            background-color: #2a2a2a;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
            padding: 20px;
            margin-top: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        #chat-box {
            width: 100%;
            height: 400px;
            overflow-y: scroll;
            border: none;
            background-color: transparent;
            color: white;
            margin-bottom: 20px;
            padding: 10px;
            text-align: right;
            font-size: 1rem; /* Default font size */
        }
        .bottom-container {
            width: 100%;
            max-width: 600px;
            display: flex;
            flex-direction: column;
            align-items: center;
            position: fixed;
            bottom: 0;
            left: 0;
            background-color: #2a2a2a;
            padding: 10px;
        }
        .input-container {
            width: 100%;
            display: flex;
            align-items: center;
            margin-bottom: 10px;
        }
        #message-input {
            width: calc(100% - 80px);
            padding: 10px;
            border: none;
            border-radius: 20px;
            background-color: #3a3a3a;
            color: white;
            outline: none;
            font-size: 1rem; /* Default font size */
        }
        .input-container button {
            padding: 10px 20px;
            border: none;
            border-radius: 20px;
            background-color: #4CAF50;
            color: white;
            cursor: pointer;
            margin-left: 10px;
            font-size: 1rem; /* Default font size */
        }
        .options-container {
            width: 100%;
            display: flex;
            justify-content: space-around;
            margin-top: 10px;
        }
        .options-container button {
            padding: 10px 20px;
            border: none;
            border-radius: 20px;
            background-color: #4CAF50;
            color: white;
            cursor: pointer;
            font-size: 1rem; /* Default font size */
        }
        @media (max-width: 768px) {
            .bottom-container {
                padding: 5px;
            }
        }
        @media (max-width: 480px) {
            .bottom-container {
                padding: 3px;
            }
        }
    </style>
</head>
<body>
    <div class="header">
        <button id="menu-button">☰</button>
        <h1>ChatAI</h1>
        <button id="settings-button">⚙️</button>
    </div>
    <div class="chat-container">
        <div id="chat-box"></div>
    </div>
    <div class="bottom-container">
        <div class="input-container">
            <input type="text" id="message-input" placeholder="اسأل عن أي شيء">
            <button onclick="sendMessage()">إرسال</button>
        </div>
        <div class="options-container">
            <button onclick="createImage()">إنشاء صورة</button>
            <button onclick="summarizeText()">لخص النص</button>
            <button onclick="getInsights()">احصل على نصائح</button>
            <button onclick="showMore()">المزيد</button>
        </div>
    </div>
    <script>
        // متغير يحتوي على مربع الدردشة
        let chatBox = document.getElementById('chat-box');
        // متغير يحتوي على المحادثة الحالية
        let currentConversation = [];

        // قاعدة بيانات الأسئلة والردود
        const qaDatabase = [
            { question: 'هلا', answer: ' اهلا وسهلا كيف يمكنني مساعدتك؟ ' },
            { question: 'هلو', answer: 'هلو كيف يمكنني مسعدتك' },
            { question: 'من صنعك', answer: 'أنا صنعت بواسطة مطور يدعى يوسف.' },
            // يمكنك إضافة المزيد من الأسئلة والردود هنا
        ];

        // دالة لإرسال الرسالة
        function sendMessage() {
            // الحصول على الرسالة من مدخل المستخدم
            let message = document.getElementById('message-input').value;
            // التحقق من أن الرسالة ليست فارغة
            if (message.trim() !== '') {
                // إضافة الرسالة إلى مربع الدردشة
                addMessage('You', message);
                // تفريغ مدخل المستخدم
                document.getElementById('message-input').value = '';
                // استدعاء الدالة للحصول على رد الذكاء الاصطناعي بعد ثانية
                setTimeout(() => {
                    getAIAutoReply(message);
                }, 1000);
            }
        }

        // دالة لإضافة الرسالة إلى مربع الدردشة
        function addMessage(sender, message) {
            // إنشاء عنصر جديد للرسالة
            let msgElement = document.createElement('div');
            // إضافة محتوى الرسالة إلى العنصر الجديد
            msgElement.innerHTML = `<strong>${sender}:</strong> ${message}`;
            // إضافة العنصر الجديد إلى مربع الدردشة
            chatBox.appendChild(msgElement);
            // التمرير إلى الأسفل في مربع الدردشة
            chatBox.scrollTop = chatBox.scrollHeight;
            // إضافة الرسالة إلى المحادثة الحالية
            currentConversation.push({ sender, message });
            // حفظ المحادثة في التخزين المحلي
            saveConversation();
        }

        // دالة لعرض الرد حرفًا حرفًا
        function typeMessage(sender, message, delay = 50) { // تقليل التأخير لتسريع الانميشن
            // إنشاء عنصر جديد للرسالة
            let msgElement = document.createElement('div');
            // إضافة اسم المرسل إلى العنصر الجديد
            msgElement.innerHTML = `<strong>${sender}:</strong>`;
            // إضافة العنصر الجديد إلى مربع الدردشة
            chatBox.appendChild(msgElement);
            // عرض الرد حرفًا حرفًا
            for (let i = 0; i < message.length; i++) {
                setTimeout(() => {
                    msgElement.innerHTML += message[i];
                    chatBox.scrollTop = chatBox.scrollHeight;
                }, i * delay);
            }
        }

        // دالة للحصول على رد الذكاء الاصطناعي
        function getAIAutoReply(message) {
            // تحويل الرسالة إلى صيغة صغيرة
            const lowerCaseMessage = message.toLowerCase();
            // البحث عن الإجابة المناسبة في قاعدة البيانات
            for (let i = 0; i < qaDatabase.length; i++) {
                if (lowerCaseMessage.includes(qaDatabase[i].question.toLowerCase())) {
                    // عرض الإجابة حرفًا حرفًا
                    typeMessage('AI', qaDatabase[i].answer);
                    return;
                }
            }
            // إذا لم يتم العثور على إجابة، عرض رسالة عدم وجود إجابة
            typeMessage('AI', 'عذرًا، لا أعرف الإجابة على هذا السؤال.');
        }

        // دالة لإنشاء صورة
        function createImage() {
            alert('Creating an image...');
        }

        // دالة لتلخيص النص
        function summarizeText() {
            alert('Summarizing the text...');
        }

        // دالة للحصول على نصائح
        function getInsights() {
            alert('Getting insights...');
        }

        // دالة لعرض المزيد من الخيارات
        function showMore() {
            alert('Showing more options...');
        }

        // دالة لفتح قائمة التاريخ
        function openHistoryMenu() {
            // تفريغ قائمة التاريخ
            historyList.innerHTML = '';
            // عرض كل محادثة في القائمة
            conversationHistory.forEach((conversation, index) => {
                let li = document.createElement('li');
                li.textContent = `سجل ${index + 1}: ${conversation[0].message}`;
                li.onclick = () => showConversation(conversation);
                historyList.appendChild(li);
            });
            // عرض قائمة التاريخ
            document.getElementById('history-menu').style.display = 'block';
        }

        // دالة لإغلاق قائمة التاريخ
        function closeHistoryMenu() {
            // إخفاء قائمة التاريخ
            document.getElementById('history-menu').style.display = 'none';
        }

        // دالة لعرض محادثة محددة
        function showConversation(conversation) {
            // تفريغ مربع الدردشة
            chatBox.innerHTML = '';
            // إضافة كل رسالة في المحادثة إلى مربع الدردشة
            conversation.forEach(msg => {
                addMessage(msg.sender, msg.message);
            });
            // تحديث المحادثة الحالية
            currentConversation = conversation;
            // إغلاق قائمة التاريخ
            closeHistoryMenu();
        }

        // دالة لحفظ المحادثة في التخزين المحلي
        function saveConversation() {
            localStorage.setItem('currentConversation', JSON.stringify(currentConversation));
        }

        // عند تحميل الصفحة، عرض المحادثة المحفوظة
        window.onload = function() {
            const storedConversation = JSON.parse(localStorage.getItem('currentConversation')) || [];
            if (storedConversation.length > 0) {
                showConversation(storedConversation);
            }
        };

        // إضافة مستمع للنقر على زر القائمة
        document.getElementById('menu-button').addEventListener('click', () => {
            // حفظ المحادثة الحالية قبل فتح قائمة التاريخ
            if (currentConversation.length > 0) {
                saveConversation();
            }
            // فتح قائمة التاريخ
            openHistoryMenu();
        });

    </script>
</body>
</html>
