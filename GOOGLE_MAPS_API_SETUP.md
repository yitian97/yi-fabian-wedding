# 如何获取 Google Maps API 密钥

本指南将帮助您设置自己的 Google Maps API 密钥，以便您的婚礼网站上的地图能够正常工作。

## 步骤 1：创建 Google Cloud 项目

1. 访问 [Google Cloud Console](https://console.cloud.google.com/)
2. 使用您的 Google 账号登录
3. 点击顶部的项目下拉菜单（"Google Cloud" 旁边）
4. 点击 **"新建项目"**
5. 输入项目名称（例如："Wedding Website"）
6. 点击 **"创建"**

## 步骤 2：启用结算（必需）

⚠️ **重要提示**：Google Maps API 需要启用结算功能，但 Google 每月会提供 $200 的免费额度，这对于婚礼网站来说通常已经足够了。

1. 在您的项目中，转到左侧菜单中的 **"结算"**
2. 点击 **"关联结算账号"**
3. 按照提示添加付款方式
4. 不用担心 - 您每月会获得 $200 的免费额度，这对于婚礼网站来说绰绰有余

## 步骤 3：启用 Maps JavaScript API

1. 在左侧菜单中，转到 **"API 和服务"** → **"库"**
2. 搜索 **"Maps JavaScript API"**
3. 点击它
4. 点击 **"启用"**
5. 等待启用完成（通常需要几秒钟）

## 步骤 4：创建 API 密钥

1. 转到 **"API 和服务"** → **"凭据"**
2. 点击顶部的 **"+ 创建凭据"**
3. 选择 **"API 密钥"**
4. 您的 API 密钥将被创建并显示
5. **复制 API 密钥** - 它看起来像这样：`AIzaSyXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`

## 步骤 5：限制您的 API 密钥（建议，用于安全）

1. 点击您新创建的 API 密钥进行编辑
2. 在 **"API 限制"** 下，选择 **"限制密钥"**
3. 勾选 **"Maps JavaScript API"**（取消勾选其他选项）
4. 在 **"应用限制"** 下，您可以选择：
   - 选择 **"HTTP 引荐来源网址（网站）"**
   - 添加您的网站 URL（例如：`https://yourdomain.com/*`）
   - 或者为了测试而保持不限制
5. 点击 **"保存"**

## 步骤 6：更新您的网站

1. 在您的项目中打开 `index.html`
2. 找到这一行（大约在第 478 行）：
   ```html
   src="https://maps.googleapis.com/maps/api/js?key=AIzaSyBlPw3GQYI3faa_9mRE6plWuM7xNEmrwH0&callback=initMap"
   ```
3. 将 `AIzaSyBlPw3GQYI3faa_9mRE6plWuM7xNEmrwH0` 替换为您的新 API 密钥
4. 保存文件

## 步骤 7：测试！

1. 刷新您的网站
2. 滚动到地图部分
3. 地图现在应该可以正常加载了！

## 故障排除

### 地图仍然无法加载？
- **检查浏览器控制台**（F12 → 控制台标签）查看错误消息
- **验证 API 密钥**在您的 HTML 中是否正确
- **检查 API 限制** - 确保 "Maps JavaScript API" 已启用
- **等待几分钟** - 有时 API 密钥需要一点时间才能激活

### "未启用结算"错误？
- 返回步骤 2 并启用结算
- 即使启用了结算，您每月仍会获得 $200 的免费额度

### "API 密钥无效"错误？
- 仔细检查您是否复制了完整的 API 密钥
- 确保没有多余的空格
- 在 Google Cloud Console 中验证 API 密钥是否已启用

### 想查看您的使用情况？
- 转到 **"API 和服务"** → **"信息中心"**
- 您可以查看您的 API 使用情况和剩余额度

## 费用信息

- **免费层级**：每月 $200 的额度
- **Maps JavaScript API**：每 1,000 次地图加载约 $7
- **对于婚礼网站**：您可能总共使用不到 $10（完全在免费层级内）

## 安全最佳实践

1. **限制您的 API 密钥**仅用于您需要的 API（Maps JavaScript API）
2. **添加 HTTP 引荐来源网址限制**以防止他人使用您的密钥
3. **不要将您的 API 密钥**提交到公共代码库
4. **定期监控使用情况**在 Google Cloud Console 中

---

**需要帮助？** 查看 [Google Maps Platform 文档](https://developers.google.com/maps/documentation/javascript/get-api-key) 或使用 [API 密钥验证工具](https://console.cloud.google.com/google/maps-apis/credentials) 测试您的 API 密钥。
