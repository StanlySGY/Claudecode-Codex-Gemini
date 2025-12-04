title
title
1908×312 172 KB
大家一定要更新gemini cli 到最新版本v0.18，修复了很多bug，可用性提高了不少！

npm install -g @google/gemini-cli
简单来说，Gemini-3p的前端效果很好（虽然后端效果一坨），但Gemini CLI的bug实在太多太多，尤其是那些内置的tool，经常一调用就会出现“API Error”，我寻思google没特训一下吗？ :distorted_face: 所以俺只能自己写个MCP让gemini充当设计师角色，而让cc来写代码了。

github.com

GitHub - GuDaStudio/geminimcp: Gemini-MCP is an MCP server that encapsulates...
Gemini-MCP is an MCP server that encapsulates Google's Gemini CLI tool into a standard MCP protocol interface, enabling Claude Code to invoke Gemini for AI-assisted programming tasks.

当然，最令我兴奋的是，在最近两天的大量开发测评中，结合第一期的CodeX-MCP，我的CC第一次有了agents的模样，基本上现在开发真的只用聊天即可，三个小agent彼此取长补短，左右互搏，看得我是不亦乐乎，最重要的是最终的代码质量出奇的高 :tieba_003:

一个示例： 本次测评使用完全相同 的提示词。使用 / 不使用本项目的效果如下所示，可以看到，使用了Gemini-MCP后的前端网页无论是在审美布局还是在动画特效上都明显优于仅使用claude code进行前端编码的版本。

 
异常简单的安装方式：

:warning: 本项目默认您已符合一下前置要求：已安装并配置 Claude Code；已安装并配置 Gemini CLI；已安装 uv 工具。

注意：强烈推荐 Windows 用户在 WSL 中运行本项目！

1.1 然后就可以通过以下命令一键为cc安装 Gemini-MCP


claude mcp add gemini -s user --transport stdio -- uvx --from git+https://github.com/GuDaStudio/geminimcp.git geminimcp

1.2 当然您可以验证安装


claude mcp list

看到 gemini: ... - ✓ Connected 说明安装成功

1.3 可选：配置自动允许

在 ~/.claude/settings.json 的 allow 项中添加 mcp__gemini__gemini

image
image
1730×877 31.8 KB
1.4 :french_fries: 我十分推荐配置的claude code提示词

在 ~/.claude/CLAUDE.md中添加以下内容

展开查看prompt
哦对了，千万别忘了最关键最重要的一步 ，快去帮我点击那个诱人的star吧！~ :lark_171:

往期回顾：

【自己动手，丰衣足食 00】 Claude和CodeX协同办公很好，我很爱，可是官方mcp写的实在太烂（已支持Windows/Linux） - 开发调优 - LINUX DO