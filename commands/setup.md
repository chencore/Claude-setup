---
argument-hint: [category|server-names]
description: 配置 MCP 服务器（可选：类别或逗号分隔的服务器名称）
---

# 配置 MCP 服务器

通过比较主配置与现有项目设置，智能管理 MCP 服务器配置，并允许选择性添加新的 MCP。

## 使用方法

```
/setup-mcp                           # 交互模式
/setup-mcp research                  # 添加所有研究类 MCP
/setup-mcp seo                       # 添加所有 SEO 类 MCP
/setup-mcp frontend                  # 添加所有前端类 MCP
/setup-mcp exa,brave-search          # 添加特定 MCP（逗号分隔）
```

## 类别

- **research**: exa, brave-search, reddit-mcp, reddit
- **seo**: dataforseo, firecrawl-mcp
- **frontend**: chrome-devtools, vibe-annotations, shadcn, next-devtools

## 描述

此命令通过以下方式为项目设置 MCP（模型上下文协议）服务器：

1. 从 `~/.claude/mcp-config.json` 读取主 MCP 配置
2. 与项目中的现有 `.mcp.json` 进行比较（如果存在）
3. 识别当前项目中不存在的新 MCP
4. 询问用户要添加哪些新 MCP
5. 添加选定的 MCP 并默认启用它们
6. 保留现有 MCP 配置及其启用/禁用状态

## 实现

```javascript
// 从 ~/.claude/mcp-config.json 读取主 MCP 配置
// 注意：readTextFile 读取整个文件内容，没有行数限制
const homeDir = Deno.env.get("HOME");
const masterConfigPath = `${homeDir}/.claude/mcp-config.json`;

let masterConfig;
try {
  // 无论大小如何，都读取完整的文件内容
  const masterConfigText = await Deno.readTextFile(masterConfigPath);
  masterConfig = JSON.parse(masterConfigText);
} catch (error) {
  console.log(`❌ Could not read master config from ${masterConfigPath}`);
  console.log("Please ensure the file exists and contains valid JSON.");
  Deno.exit(1);
}

// 检查项目中是否存在 .mcp.json
// 注意：readTextFile 读取整个文件内容，没有行数限制
let existingConfig = { mcpServers: {} };
let existingMcpNames = [];
try {
  // 无论大小如何，都读取完整的文件内容
  const existingConfigText = await Deno.readTextFile(".mcp.json");
  existingConfig = JSON.parse(existingConfigText);
  existingMcpNames = Object.keys(existingConfig.mcpServers || {});
} catch {
  // 没有现有配置，这没问题
}

// 定义类别映射
const categories = {
  research: ["exa", "brave-search", "reddit-mcp", "reddit"],
  seo: ["dataforseo", "firecrawl-mcp"],
  frontend: ["chrome-devtools", "vibe-annotations", "shadcn", "next-devtools"]
};

// 从 $ARGUMENTS 获取参数
const argument = "$ARGUMENTS".trim();

// 查找现有配置中不存在的新 MCP
const masterMcpNames = Object.keys(masterConfig.mcpServers);
const newMcpNames = masterMcpNames.filter(
  (name) => !existingMcpNames.includes(name)
);

if (newMcpNames.length === 0) {
  console.log(
    "✅ All MCPs from master config are already present in this project."
  );
  console.log(`Current MCPs: ${existingMcpNames.join(", ")}`);
  Deno.exit(0);
}

let selectedMcps = [];

// 处理基于参数的选择
if (argument) {
  // 检查是否为类别
  if (categories[argument]) {
    selectedMcps = categories[argument].filter(name => newMcpNames.includes(name));
    if (selectedMcps.length === 0) {
      console.log(`ℹ️  All MCPs in category '${argument}' are already installed.`);
      Deno.exit(0);
    }
    console.log(`📦 Adding ${selectedMcps.length} MCP(s) from category '${argument}': ${selectedMcps.join(", ")}`);
  } else {
    // 视为逗号分隔的服务器名称
    const requestedNames = argument.split(",").map(s => s.trim());
    selectedMcps = requestedNames.filter(name => {
      if (!masterMcpNames.includes(name)) {
        console.log(`⚠️  Warning: '${name}' not found in master config`);
        return false;
      }
      if (existingMcpNames.includes(name)) {
        console.log(`ℹ️  '${name}' is already installed`);
        return false;
      }
      return true;
    });

    if (selectedMcps.length === 0) {
      console.log("❌ No valid MCPs to add.");
      Deno.exit(0);
    }
    console.log(`📦 Adding ${selectedMcps.length} MCP(s): ${selectedMcps.join(", ")}`);
  }
} else {
  // 交互模式 - 询问用户要添加哪些 MCP
  console.log(`\n📋 Found ${newMcpNames.length} new MCP(s) available:`);
  newMcpNames.forEach((name, index) => {
    console.log(`${index + 1}. ${name}`);
  });

  console.log("\nWhich MCPs would you like to add?");
  console.log("- Type 'all' to add all new MCPs");
  console.log("- Type specific numbers (e.g., '1,3,5') to add selected MCPs");
  console.log("- Type 'none' to cancel");

  const input = prompt("Your choice: ");

  if (input === "all") {
    selectedMcps = [...newMcpNames];
  } else if (input === "none" || !input) {
    console.log("❌ Operation cancelled.");
    Deno.exit(0);
  } else {
    // 解析逗号分隔的数字
    const indices = input.split(",").map((s) => parseInt(s.trim()) - 1);
    selectedMcps = indices
      .filter((i) => i >= 0 && i < newMcpNames.length)
      .map((i) => newMcpNames[i]);
  }
}

if (selectedMcps.length === 0) {
  console.log("❌ No valid MCPs selected.");
  Deno.exit(0);
}

// 将选定的 MCP 添加到现有配置
for (const mcpName of selectedMcps) {
  existingConfig.mcpServers[mcpName] = masterConfig.mcpServers[mcpName];
}

// 创建 .claude 目录（如果不存在）
await Deno.mkdir(".claude", { recursive: true });

// 写入更新的 .mcp.json
await Deno.writeTextFile(".mcp.json", JSON.stringify(existingConfig, null, 2));

// 处理 settings.local.json
let settings = {};
try {
  const existingSettings = await Deno.readTextFile(
    ".claude/settings.local.json"
  );
  settings = JSON.parse(existingSettings);
} catch {
  // 文件不存在，从空设置开始
}

// 确保 enableAllProjectMcpServers 设置为 false
settings.enableAllProjectMcpServers = false;

// 初始化 enabledMcpjsonServers（如果不存在）
if (!settings.enabledMcpjsonServers) {
  settings.enabledMcpjsonServers = [];
}

// 初始化 disabledMcpjsonServers（如果不存在）
if (!settings.disabledMcpjsonServers) {
  settings.disabledMcpjsonServers = [];
}

// 将新选择的 MCP 添加到启用列表
for (const mcpName of selectedMcps) {
  if (!settings.enabledMcpjsonServers.includes(mcpName)) {
    settings.enabledMcpjsonServers.push(mcpName);
  }
}

await Deno.writeTextFile(
  ".claude/settings.local.json",
  JSON.stringify(settings, null, 2)
);

console.log(
  `\n✅ Added ${selectedMcps.length} new MCP(s): ${selectedMcps.join(", ")}`
);
console.log("✅ New MCPs added to enabledMcpjsonServers list");
console.log("✅ Existing MCP configurations preserved");

console.log("\n🔄 Please restart Claude Code for the changes to take effect!");
```
