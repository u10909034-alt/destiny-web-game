# destiny-web-game
# Let's create an example Visual Novel HTML/CSS/JS game that the user can immediately host on GitHub Pages.
# It will load local images (which the user downloaded to their computer) and have a visual novel style (text box, choices, background).
# Since WeasyPrint is for PDF, wait, the user is NOT asking for a PDF or Excel. They are asking for how to do this.
# But wait, as a developer, if I can write a complete, self-contained `index.html` file that they can download and put in their folder along with their images, that would be incredibly helpful!
# Let's generate a beautiful HTML/CSS/JS single-file game template (index.html) and package it as a downloadable ZIP or text file, or just present the code and generate a ZIP/HTML file for them to download.
# Let's write a Python script to package a template ZIP containing `index.html` and a detailed `README.md` explaining how to place their Canva images and upload it to GitHub.

import zipfile
import os

html_content = """<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的文字冒險遊戲</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }
        body, html {
            width: 100%;
            height: 100%;
            font-family: 'Microsoft JhengHei', sans-serif;
            background-color: #000;
            overflow: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        #game-container {
            position: relative;
            width: 100%;
            max-width: 960px;
            height: 100%;
            max-height: 540px;
            aspect-ratio: 16 / 9;
            background-color: #111;
            background-size: cover;
            background-position: center;
            transition: background-image 0.5s ease-in-out;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
            border: 2px solid #333;
            border-radius: 8px;
            overflow: hidden;
        }
        /* 故事文字與對話框 */
        #textbox-container {
            position: absolute;
            bottom: 20px;
            left: 5%;
            width: 90%;
            height: 140px;
            background: rgba(20, 20, 20, 0.85);
            border: 2px solid rgba(255, 255, 255, 0.3);
            border-radius: 10px;
            padding: 15px 25px;
            color: #fff;
            cursor: pointer;
            z-index: 10;
        }
        #speaker-name {
            font-size: 18px;
            font-weight: bold;
            color: #ffd700;
            margin-bottom: 8px;
            text-shadow: 1px 1px 2px #000;
        }
        #dialogue-text {
            font-size: 16px;
            line-height: 1.6;
            letter-spacing: 1px;
        }
        #click-prompt {
            position: absolute;
            right: 15px;
            bottom: 10px;
            font-size: 12px;
            color: #aaa;
            animation: blink 1.5s infinite;
        }
        @keyframes blink {
            0%, 100% { opacity: 0; }
            50% { opacity: 1; }
        }
        /* 選項選單 */
        #choices-container {
            position: absolute;
            top: 40%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 80%;
            display: flex;
            flex-direction: column;
            gap: 15px;
            z-index: 20;
        }
        .choice-btn {
            background: rgba(0, 0, 0, 0.85);
            color: #fff;
            border: 2px solid #ffd700;
            padding: 12px 20px;
            font-size: 16px;
            border-radius: 25px;
            cursor: pointer;
            text-align: center;
            transition: all 0.2s ease;
            box-shadow: 0 4px 6px rgba(0,0,0,0.3);
        }
        .choice-btn:hover {
            background: #ffd700;
            color: #000;
            transform: scale(1.02);
            font-weight: bold;
        }
        /* 開始畫面 */
        #start-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.9);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 100;
            color: white;
        }
        #start-title {
            font-size: 36px;
            margin-bottom: 30px;
            color: #ffd700;
            letter-spacing: 2px;
        }
        .start-btn {
            padding: 12px 40px;
            font-size: 18px;
            background: transparent;
            color: #ffd700;
            border: 2px solid #ffd700;
            border-radius: 5px;
            cursor: pointer;
            transition: all 0.3s ease;
        }
        .start-btn:hover {
            background: #ffd700;
            color: #000;
        }
    </style>
</head>
<body>

    <div id="game-container">
        <div id="start-screen">
            <h1 id="start-title">橙光風文字冒險遊戲</h1>
            <button class="start-btn" onclick="startGame()">開始遊戲</button>
        </div>

        <div id="choices-container" style="display: none;"></div>

        <div id="textbox-container" onclick="nextStep()" style="display: none;">
            <div id="speaker-name">角色名字</div>
            <div id="dialogue-text">正在加載遊戲內容...</div>
            <div id="click-prompt">▼ 點擊繼續</div>
        </div>
    </div>

    <script>
        // 【自訂劇本與圖片設定區】
        // 請將你從 Canva 下載的圖片重新命名（例如 1.png, 2.png, 3.png 等）
        // 並放入與 index.html 相同的資料夾（或建立一個名為 images 的資料夾）
        const scriptData = {
            "start": {
                bg: "images/1.png", // 第一張 Canva 圖片路徑
                speaker: "旁白",
                text: "你在一個陌生的地方醒來，四周一片寂靜...",
                next: "scene1_choice"
            },
            "scene1_choice": {
                bg: "images/1.png",
                speaker: "旁白",
                text: "前方有一扇古老的門，和一條通往地下的樓梯。你要選擇哪裡？",
                choices: [
                    { text: "打開古老的門", target: "door_path" },
                    { text: "走下地下樓梯", target: "stairs_path" }
                ]
            },
            "door_path": {
                bg: "images/2.png", // 第二張 Canva 圖片路徑
                speaker: "神祕人",
                text: "「你終於來了... 我等了你很久。」這扇門後竟然站著一個人！",
                next: "game_end_1"
            },
            "stairs_path": {
                bg: "images/3.png", // 第三張 Canva 圖片路徑
                speaker: "主角",
                text: "地下室一片漆黑，我隱約聽見了水滴聲... 好像有什麼不妙的事要發生。",
                next: "game_end_2"
            },
            "game_end_1": {
                bg: "images/2.png",
                speaker: "結局一",
                text: "【命運的重逢】你成功與神祕人會合，解開了這個世界的謎團！(遊戲結束，感謝遊玩)",
                next: "title_screen"
            },
            "game_end_2": {
                bg: "images/3.png",
                speaker: "結局二",
                text: "【黑暗深淵】你消失在無盡的地下迷宮中，再也沒有人見過你... (遊戲結束，感謝遊玩)",
                next: "title_screen"
            }
        };

        let currentKey = "start";
        const gameContainer = document.getElementById("game-container");
        const startScreen = document.getElementById("start-screen");
        const textboxContainer = document.getElementById("textbox-container");
        const choicesContainer = document.getElementById("choices-container");
        const speakerName = document.getElementById("speaker-name");
        const dialogueText = document.getElementById("dialogue-text");

        function startGame() {
            startScreen.style.display = "none";
            textboxContainer.style.display = "block";
            currentKey = "start";
            renderScene();
        }

        function renderScene() {
            const node = scriptData[currentKey];
            if (!node) return;

            // 設定背景圖片 (如果沒有設定，就維持原本的)
            if (node.bg) {
                gameContainer.style.backgroundImage = `url('${node.bg}')`;
            }

            // 設定說話者與對話
            speakerName.innerText = node.speaker || "";
            dialogueText.innerText = node.text || "";

            // 隱藏選項，直到點擊或需要展示時
            choicesContainer.style.display = "none";
            choicesContainer.innerHTML = "";

            // 如果該節點有選擇肢
            if (node.choices) {
                textboxContainer.style.pointerEvents = "none"; // 點擊對話框不會觸發下一步
                document.getElementById("click-prompt").style.display = "none";
                
                node.choices.forEach(choice => {
                    const btn = document.createElement("button");
                    btn.className = "choice-btn";
                    btn.innerText = choice.text;
                    btn.onclick = () => {
                        currentKey = choice.target;
                        textboxContainer.style.pointerEvents = "auto";
                        document.getElementById("click-prompt").style.display = "block";
                        renderScene();
                    };
                    choicesContainer.appendChild(btn);
                });
                
                // 延遲一點點顯示選項，讓玩家看完字
                setTimeout(() => {
                    choicesContainer.style.display = "flex";
                }, 300);
            }
        }

        function nextStep() {
            const node = scriptData[currentKey];
            if (!node) return;

            // 如果目前有選擇肢，點擊對話框無效
            if (node.choices) return;

            if (node.next) {
                if (node.next === "title_screen") {
                    // 回到標題畫面
                    startScreen.style.display = "flex";
                    textboxContainer.style.display = "none";
                } else {
                    currentKey = node.next;
                    renderScene();
                }
            }
        }
    </script>
</body>
</html>
"""

readme_content = """# 橙光風格文字冒險遊戲網頁範本

恭喜你！這是一個專門為你設計的網頁文字冒險遊戲範本。你不需要上傳圖片到別的網站，只需要將這個專案資料夾放到你電腦裡，並跟著以下步驟，就能在 GitHub Pages 上免費發布！

## 📁 檔案結構準備

在你的電腦上建立一個資料夾（例如 `my-game`），並確保裡面的檔案結構如下：
