# Termux-boost-
no root 

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Command & Code Repository</title>
    <!-- Font Awesome Icons for the copy button -->
    <link rel="stylesheet" href="https://cloudflare.com">
    
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f6f8fa;
            padding: 20px;
            color: #24292f;
        }
        .code-container {
            background-color: #ffffff;
            border: 1px solid #d0d7de;
            border-radius: 6px;
            padding: 15px;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            max-width: 600px;
        }
        .copy-btn {
            background: none;
            border: none;
            cursor: pointer;
            color: #57606a;
            font-size: 18px;
            margin-right: 15px; /* Places the button in front of the text */
            transition: color 0.2s;
        }
        .copy-btn:hover {
            color: #0969da; /* Changes to blue on hover */
        }
        .code-text {
            font-family: monospace;
            background-color: #afb8c133;
            padding: 3px 6px;
            border-radius: 6px;
            font-size: 14px;
        }
    </style>
</head>
<body>

    <h1>📋 My Command Library</h1>
    <p>Click the icon in front of any command to copy it to your clipboard.</p>

    <!-- Command Item 1 -->
    <div class="code-container">
        <button class="copy-btn" onclick="copyToClipboard('code1', this)" title="Copy Command">
            <i class="fa-regular fa-copy"></i>
        </button>
        <span class="code-text" id="code1">git clone https://github.com</span>
    </div>

    <!-- Command Item 2 -->
    <div class="code-container">
        <button class="copy-btn" onclick="copyToClipboard('code2', this)" title="Copy Command">
            <i class="fa-regular fa-copy"></i>
        </button>
        <span class="code-text" id="code2">npm install react</span>
    </div>

    <!-- JavaScript Function for Clipboard Copying -->
    <script>
        function copyToClipboard(elementId, buttonElement) {
            // Get the text from the target element ID
            const textToCopy = document.getElementById(elementId).innerText;
            
            // Use Clipboard API to copy text
            navigator.clipboard.writeText(textToCopy).then(() => {
                // Change icon to a green checkmark on success
                const icon = buttonElement.querySelector('i');
                icon.className = 'fa-solid fa-check';
                icon.style.color = '#1a7f37'; 
                
                // Revert back to the copy icon after 2 seconds
                setTimeout(() => {
                    icon.className = 'fa-regular fa-copy';
                    icon.style.color = '';
                }, 2000);
            }).catch(err => {
                console.error('Failed to copy text: ', err);
            });
        }
    </script>

</body>
</html>
