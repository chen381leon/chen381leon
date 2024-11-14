<!-- 特色部分的图标 -->
<div class="features-grid">
    <div class="feature-item">
        <img src="图标/塔罗2.png" alt="专业讲师" class="feature-icon">
        <span>专业讲师团队</span>
    </div>
    <div class="feature-item">
        <img src="图标/塔罗3.png" alt="课程体系" class="feature-icon">
        <span>系统化的课程体系</span>
    </div>
    <div class="feature-item">
        <img src="图标/塔罗4.png" alt="实践机会" class="feature-icon">
        <span>丰富的实践机会</span>
    </div>
    <div class="feature-item">
        <img src="图标/塔罗5.png" alt="AI系统" class="feature-icon">
        <span>AI辅助学习系统</span>
    </div>
</div>

<!-- 课程部分的图片 -->
<!-- 基础班 -->
<div id="basic" class="course-section my-5">
    <div class="course-header">
        <img src="图标/塔罗6.png" alt="基础课程" class="course-image">
        <h2>基础班</h2>
    </div>
    <!-- ... -->
</div>

<!-- 进阶班 -->
<div id="intermediate" class="course-section my-5">
    <div class="course-header">
        <img src="图标/塔罗7.png" alt="进阶课程" class="course-image">
        <h2>进阶班</h2>
    </div>
    <!-- ... -->
</div>

<!-- 终极班 -->
<div id="advanced" class="course-section my-5">
    <div class="course-header">
        <img src="图标/塔罗8.png" alt="终极课程" class="course-image">
        <h2>终极班</h2>
    </div>
    <!-- ... -->
    /* 调整特色图标样式 */
.feature-icon {
    width: 80px;
    height: 80px;
    object-fit: contain;  /* 改为 contain 以保持图片比例 */
    border-radius: 50%;
    margin-bottom: 15px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    transition: transform 0.3s ease;
    background-color: white;  /* 添加白色背景 */
    padding: 10px;  /* 添加内边距 */
}
// 添加对话次数计数器
let questionCount = 0;
const MAX_QUESTIONS = 3;

// 修改 sendMessage 函数
async function sendMessage() {
    const input = document.getElementById('userInput');
    const message = input.value.trim();
    
    // 检查是否达到问题限制
    if (questionCount >= MAX_QUESTIONS) {
        addMessage('ai', '您已达到免费咨询次数限制（3次）。如需继续咨询，请报名我们的课程获取更多指导。');
        return;
    }

    if (message) {
        addMessage('user', message);
        input.value = '';
        questionCount++; // 增加问题计数

        // 显示剩余次数
        const remainingQuestions = MAX_QUESTIONS - questionCount;
        addMessage('ai', `您还有 ${remainingQuestions} 次免费咨询机会。`);
        
        try {
            // 显示加载状态
            const loadingDiv = document.createElement('div');
            loadingDiv.className = 'message ai-message';
            loadingDiv.textContent = '正在思考...';
            document.getElementById('chatMessages').appendChild(loadingDiv);
            
            // 调用 Moonshot API
            const response = await fetch('https://api.moonshot.cn/v1/chat/completions', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': 'Bearer sk-IhTksniKGBkHo02MBjH850GF8qbqfwySqRjMyJLvsunRlzfA'
                },
                body: JSON.stringify({
                    model: "moonshot-v1-32k",
                    messages: [
                        {
                            role: "system",
                            content: "你是一个专业的塔罗牌解读师，请用简短的语言回答用户的问题。回答要包含塔罗牌的专业知识。每次回答限制在100字以内。"
                        },
                        {
                            role: "user",
                            content: message
                        }
                    ],
                    temperature: 0.7,
                    max_tokens: 800,
                    stream: false
                })
            });

            // 移除加载状态
            loadingDiv.remove();

            if (response.ok) {
                const data = await response.json();
                const aiResponse = data.choices[0].message.content;
                addMessage('ai', aiResponse);

                // 如果是最后一次提问，添加提示信息
                if (questionCount === MAX_QUESTIONS) {
                    setTimeout(() => {
                        addMessage('ai', '这是您的最后一次免费咨询。如需继续咨询，请报名我们的课程获取更多专业指导。');
                    }, 1000);
                }
            } else {
                throw new Error('API 请求失败');
            }
        } catch (error) {
            console.error('Error:', error);
            // 如果 API 调用失败，使用预设回复
            const tarotResponses = [
                "让我抽取一张塔罗牌为您解读。我看到了正位星星牌，这表示希望、灵感和创造力。现在是追求梦想的好时机。",
                "塔罗牌显示的是正位命运之轮，说明您正处在一个转折点。机会就在眼前，要积极把握。",
                "我为您抽到了正位力量牌，这表示您内心充满勇气和决心。现在正是克服困难的最佳时机。",
                "塔罗牌显示的是正位太阳牌，预示着光明、快乐和成功。您的付出将得到回报。",
                "我看到了正位女祭司牌，这提醒您要相信直觉，倾听内心的声音。神秘的力量正在指引着您。",
                "塔罗牌显示的是正位皇后牌，象征着丰盛、创造力和关怀。您将获得意想不到的支持。",
                "为您抽到了正位战车牌，这预示着胜利和进步。只要坚持目标，一定能取得成功。"
            ];
            const randomResponse = tarotResponses[Math.floor(Math.random() * tarotResponses.length)];
            addMessage('ai', randomResponse);
        }
    }
}

// 在页面加载时添加提示信息
document.addEventListener('DOMContentLoaded', function() {
    // ... 其他初始化代码 ...
    
    // 添加免费咨询次数提示
    const chatMessages = document.getElementById('chatMessages');
    chatMessages.innerHTML = `
        <div class="message ai-message">
            欢迎使用AI塔罗咨询！您有3次免费咨询机会。
        </div>
    `;
});
</div>
<img width="151" alt="塔罗8" src="https://github.com/user-attachments/assets/d07dbedf-ca5d-4317-83ca-ba4749244b91">
<img width="308" alt="塔罗7" src="https://github.com/user-attachments/assets/48ecc5c0-ffed-4af5-84b0-d58c20f56258">
<img width="175" alt="塔罗6" src="https://github.com/user-attachments/assets/8fd7bde9-3acb-4a95-9741-510ef1a9c158">
<img width="278" alt="塔罗5" src="https://github.com/user-attachments/assets/2adb72f8-5d5b-408f-a489-b68757255637">
<img width="265" alt="塔罗4" src="https://github.com/user-attachments/assets/7e85373f-7580-4d7c-a144-e43997f1a53d">
<img width="278" alt="塔罗3" src="https://github.com/user-attachments/assets/db5e72bd-43f9-401f-a388-08915dba729e">
<img width="155" alt="塔罗2" src="https://github.com/user-attachments/assets/1922eac9-0119-4e63-9483-f0fa0d3e235d">
<img width="152" alt="塔罗1" src="https://github.com/user-attachments/assets/e8b31e6a-3ce1-4b78-9a2d-27ec7a58567f">
![支付宝收款码](https://github.com/user-attachments/assets/c66a615d-0ca6-4dca-b8c0-630e4e7fcb2d)
![微信支付](https://github.com/user-attachments/assets/b9ece58b-5670-403e-ae61-31a032fe8f20)
![微信二维码](https://github.com/user-attachments/assets/56d80b70-3ddf-4cb8-ace5-28a7a4b479fb)

