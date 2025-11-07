---
title: 【CS日志】Typecho CDN过期修复日志
published: 2025-11-07
description: ''
image: ''
tags:
  - journal
category: CS日志
draft: false
lang: ''
---

由于很久没看双人博客了，今天室友说样式炸了。回顾了Brave主题作者的评论区，有人说是CDN炸了，bootstrap.min.css样式链接挂了，在head.php中更新一版就可以了。

打开开发者工具控制台，输入如下脚本：
```bash
// Bootstrap CSS 加载完整诊断脚本
(async function() {
    console.log('%c=== Bootstrap CSS 加载诊断 ===', 'font-size: 16px; font-weight: bold; color: #2196F3;');
    console.log('');
    
    // 1. 检查 HTML 源码中的 Bootstrap 链接
    console.log('%c1. 检查 HTML 源码中的 Bootstrap 链接', 'font-size: 14px; font-weight: bold;');
    const htmlSource = document.documentElement.outerHTML;
    const bytecdnMatch = htmlSource.match(/lf3-cdn-tos\.bytecdntp\.com[^"'\s]*bootstrap[^"'\s]*/i);
    const jsdelivrMatch = htmlSource.match(/cdn\.jsdelivr\.net[^"'\s]*bootstrap[^"'\s]*/i);
    
    if (bytecdnMatch) {
        console.error('   ❌ 发现旧的字节跳动 CDN:', bytecdnMatch[0]);
        console.error('   ⚠️  问题：文件修改可能未生效或未上传到服务器！');
    } else {
        console.log('   ✓ 未发现旧的字节跳动 CDN');
    }
    
    if (jsdelivrMatch) {
        console.log('   ✓ 发现 jsDelivr CDN:', jsdelivrMatch[0]);
    } else {
        console.warn('   ⚠️  未发现 jsDelivr CDN');
    }
    console.log('');
    
    // 2. 检查 DOM 中的样式表链接
    console.log('%c2. 检查 DOM 中的样式表链接', 'font-size: 14px; font-weight: bold;');
    const cssLinks = Array.from(document.querySelectorAll('link[rel="stylesheet"]'));
    console.log(`   找到 ${cssLinks.length} 个样式表:`);
    
    let bootstrapFound = false;
    let oldCDNFound = false;
    
    cssLinks.forEach((link, i) => {
        const href = link.href;
        const isBootstrap = /bootstrap/i.test(href);
        const isOldCDN = /bytecdntp|lf3-cdn-tos/i.test(href);
        const isNewCDN = /cdn\.jsdelivr\.net.*bootstrap/i.test(href);
        
        let status = '';
        if (isBootstrap) {
            bootstrapFound = true;
            if (isOldCDN) {
                status = ' ❌ 旧的字节跳动 CDN';
                oldCDNFound = true;
            } else if (isNewCDN) {
                status = ' ✓ jsDelivr CDN';
            } else {
                status = ' ⚠️  其他 Bootstrap 源';
            }
        }
        
        console.log(`   ${i+1}. ${href}${status}`);
        if (isBootstrap) {
            console.log(`      - Integrity: ${link.integrity || '未设置'}`);
            console.log(`      - CrossOrigin: ${link.crossOrigin || '未设置'}`);
        }
    });
    
    if (!bootstrapFound) {
        console.error('   ❌ 未找到 Bootstrap CSS 链接！');
    } else if (oldCDNFound) {
        console.error('   ❌ 发现旧的 CDN，需要更新文件！');
    } else {
        console.log('   ✓ Bootstrap CSS 链接正确');
    }
    console.log('');
    
    // 3. 检查 Bootstrap 样式是否应用
    console.log('%c3. 检查 Bootstrap 样式是否应用', 'font-size: 14px; font-weight: bold;');
    const testContainer = document.createElement('div');
    testContainer.className = 'container';
    testContainer.style.position = 'absolute';
    testContainer.style.left = '-9999px';
    document.body.appendChild(testContainer);
    
    const styles = window.getComputedStyle(testContainer);
    const hasBootstrap = styles.maxWidth !== 'none' && styles.maxWidth !== '';
    
    if (hasBootstrap) {
        console.log('   ✓ Bootstrap 样式已应用');
        console.log(`   Container max-width: ${styles.maxWidth}`);
    } else {
        console.error('   ❌ Bootstrap 样式未应用');
        console.error('   Container max-width:', styles.maxWidth);
    }
    
    document.body.removeChild(testContainer);
    console.log('');
    
    // 4. 诊断总结
    console.log('%c=== 诊断总结 ===', 'font-size: 16px; font-weight: bold; color: #4CAF50;');
    
    if (oldCDNFound) {
        console.error('   ❌ 发现旧的字节跳动 CDN');
        console.log('');
        console.log('   🔧 修复步骤:');
        console.log('   1. 检查 Typecho 后台 → 外观 → 设置 → "头部自定义"选项');
        console.log('      如果那里有旧的 CDN 链接，请删除它');
        console.log('   2. 确认 head.php 文件已上传到服务器');
        console.log('   3. 清除服务器缓存（重启 PHP-FPM）');
    } else if (!hasBootstrap) {
        console.error('   ❌ Bootstrap 样式未应用');
        console.log('   请检查 Network 标签中 bootstrap.min.css 的状态码');
    } else {
        console.log('   ✓ 未发现明显问题');
    }
    
    console.log('');
    console.log('%c=== 诊断完成 ===', 'font-size: 16px; font-weight: bold; color: #2196F3;');
})();
```

检测到诊断显示仍在使用旧的字节跳动 CDN。改了head.php也没用（难道是需要上传步骤？我好久没用Typecho了忘记了要不要传）

最后打开typecho控制台（https://<你的域名>/admin/） -> 控制台 -> 外观 -> 设置外观 -> 下拉找到“头部自定义内容”,添加一下内容（直接贴给head.php，这样就不需要研究什么重传了，懒人方法）
```php
 <link href="https://cdn.jsdelivr.net/npm/bootstrap@4.6.1/dist/css/bootstrap.min.css" type="text/css" rel="stylesheet" crossorigin="anonymous" integrity="sha384-zCbKRCUGaJDkqS1kPbPd7TveP5iyJE0EjAuZQTgFLD2ylzuqKfdKlfG/eSrtxUkn" />
   <script src="https://cdn.jsdelivr.net/npm/jquery@3.6.0/dist/jquery.min.js" type="application/javascript" crossorigin="anonymous"></script>
   <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.6.1/dist/js/bootstrap.bundle.min.js" type="application/javascript" crossorigin="anonymous"></script>
```