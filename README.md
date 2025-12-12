# Auto-review

## 起因：
  期末了，学校又开始了烦人的评价教师环节。
  <br>
  一个学期动辄十多二十个老师，每个老师评分都得手动输入分数
  <br>
  而且还不能全部给一样的分数。于是便有了这个。
  <br>
  ⚠️仅自用，我的coding skill很差，不要指望我能写出什么好东西（（

## 效果：
- 自动评价教师
- 随机一个选项给予9分，其余全部满分
- 只需复制粘贴即可，无须多余操作
- （编不出来了，就这样吧）

## 使用方法：
  复制以下内容，打开学校教务评教页面
  ~~~
(async function () {
    const INPUT_TIMEOUT_MS = 8000;      // 评分输入框查找容错时间 (8秒)
    const NEXT_REVIEW_WAIT_MS = 4000;   // 提交后等待进入下一轮的时间 (4秒)
    const INTERVAL_MS = 200;            // 查找间隔
    
    // --- 辅助函数：等待指定毫秒 ---
    const wait = (ms) => new Promise(res => setTimeout(res, ms));

    // --- 辅助函数：等待元素出现 ---
    async function waitForElement(selector, timeout, interval = INTERVAL_MS, isList = false) {
        const startTime = Date.now();
        while (Date.now() - startTime < timeout) {
            let elements = Array.from(document.querySelectorAll(selector));

            if (selector.includes('input')) {
                elements = elements.filter(i => 
                    !i.readOnly && 
                    i.closest('li.clearfix') && 
                    i.closest('li.clearfix').querySelector('span.btbs')
                );
            }

            if ((isList && elements.length > 0) || (!isList && elements.length > 0 && !isList)) {
                return isList ? elements : elements[0];
            }
            await wait(interval);
        }
        return isList ? [] : null;
    }
    
    // --- 模拟输入 ---
    function realInput(el, value) {
        if (el.readOnly) return;
        el.value = value;
        const event = new Event('input', { bubbles: true });
        el.dispatchEvent(event);
        el.dispatchEvent(new Event('change', { bubbles: true }));
    }

    console.log("🚀 评教已启动");

    // 1. 主循环，直到所有“未评价”的行都被处理
    while (true) {
        await wait(500); // 确保列表稳定

        // 查找所有“未评价”的行
        const reviewRows = Array.from(document.querySelectorAll("tr.el-table__row"))
            .filter(r => r.querySelector(".wpj")?.innerText.includes("未评价"));

        if (reviewRows.length === 0) {
            console.log("\n✅ 所有老师均已评价完成！程序结束。");
            break;
        }

        const totalRemaining = reviewRows.length;
        const row = reviewRows[0]; // 总是处理列表中第一个“未评价”的老师
        const name = row.querySelector(".el-table_1_column_3 .cell")?.innerText.trim() || `教师 ${1}`;

        console.log(`\n👉 (剩余 ${totalRemaining} 位) 开始评价：${name}`);

        // 2. 点击评价按钮
        const reviewBtn = row.querySelector(".btn_theme");
        if (!reviewBtn) {
            console.error(`❌ 找不到 ${name} 的评价按钮，本次跳过。`);
            await wait(NEXT_REVIEW_WAIT_MS); 
            continue;
        }

        reviewBtn.click();
        
        // 3. 等待评分输入框出现 (等待 8s)
        await wait(500); // 初始等待
        const inputs = await waitForElement(".el-dialog__wrapper.whole input", INPUT_TIMEOUT_MS, INTERVAL_MS, true); 

        const groupSize = inputs.length; 
        if (groupSize === 0) {
            console.warn(`⚠️ 找不到 ${name} 的评分输入框（${INPUT_TIMEOUT_MS / 1000}秒超时），跳过.`);
            document.querySelector(".el-dialog__headerbtn")?.click(); 
            await wait(NEXT_REVIEW_WAIT_MS); 
            continue;
        }
        
        // 4. 输入评分
        const nineIndex = Math.floor(Math.random() * groupSize);
        inputs.forEach((input, i) => {
            const score = i === nineIndex ? 9 : 10;
            realInput(input, score);
        });

        console.log(`✔ 分数模拟完成，共 ${groupSize} 项。随机给第 ${nineIndex + 1} 项打了 9 分。`);
        await wait(500); 

        // 5. 点击“提交”按钮
        const submitBtn = document.querySelector(".el-dialog__header button.theme_color.btn");
        if (submitBtn) {
            submitBtn.click();
            console.log("✔ 已点击提交按钮");
        } else {
            console.error("❌ 找不到提交按钮！跳过本轮评价.");
            document.querySelector(".el-dialog__headerbtn")?.click(); 
            await wait(NEXT_REVIEW_WAIT_MS);
            continue;
        }

        // 7. 贤者时间（防止速度过快表单的提交按钮消失）
        console.log(`⏸ 等待 ${NEXT_REVIEW_WAIT_MS / 1000} 秒后继续...`);
        await wait(NEXT_REVIEW_WAIT_MS); 
        
        document.querySelector(".el-dialog__headerbtn")?.click();
    }
})();

~~~
