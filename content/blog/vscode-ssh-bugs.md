---
title: "VS Code - SSH bugs"
description: "Using VsCode to connect to your remote?  Enjoy ever escalating CPU and disk reads until it crashes."
date: "2022-05-26"
timezone: "Europe/London"
utterances_term: "Hello World"
categories: ["VsCode", "SSH"]
---

# VsCode and SSH

Are you suing VsCode to connect to your remote machine?
Find it slowing down, until the remote stops responding, then crashes completely?
Seeing monitoring like this?
![DigitalOcean Monitoring - Disk thrashing](https://user-images.githubusercontent.com/72463218/170488097-84ced8c1-3994-46dc-a4b8-616f9b96c79d.png)

Try either disabling "TypeScript and Javascript Language Features extension" when you're connecting in your main VS code.
(Thanks to [Ben Olayinka](https://medium.com/good-robot/use-visual-studio-code-remote-ssh-sftp-without-crashing-your-server-a1dc2ef0936d))

Or install "VsCode Insider Edition" to reserve for your SSH work, and disable troublesome extensions in that to preserve your main development tool.
(Thanks [@prashant9912](https://github.com/prashant9912))

Maybe come jump on the GitHub Issue, and see if we can persuade someone to fix this?
[https://github.com/microsoft/vscode-remote-release/issues/2692#issuecomment-1136321906](https://github.com/microsoft/vscode-remote-release/issues/2692#issuecomment-1136321906)