# FitTrack 健身动作库

单文件 HTML 健身应用，支持动作库浏览、训练计划、计时器、云端同步。

## 文件说明

| 文件 | 说明 |
|------|------|
| `index.html` | 应用主文件 |
| `config.js` | Supabase 配置文件（部署前修改） |
| `.nojekyll` | GitHub Pages 必需，禁用 Jekyll 处理 |
| `.github/workflows/keep-supabase-alive.yml` | GitHub Actions 自动保活脚本，防止项目被暂停 |

## 本地运行

直接用浏览器打开 `index.html` 即可，或在项目目录运行：

```bash
python3 -m http.server 8099
```

然后访问 `http://localhost:8099`

## 部署到 GitHub Pages

1. 创建 GitHub 仓库（如 `fittrack`）
2. 将所有文件推送到仓库：
   ```bash
   git init
   git add .
   git commit -m "FitTrack 初始部署"
   git branch -M main
   git remote add origin https://github.com/你的用户名/fittrack.git
   git push -u origin main
   ```
3. 进入仓库 **Settings → Pages**
4. Source 选择 **Deploy from a branch**
5. Branch 选择 `main`，文件夹选 `/ (root)`
6. 点击 **Save**
7. 等待 1-2 分钟，访问 `https://你的用户名.github.io/fittrack/`

## 自定义 Supabase 配置

1. 注册 [Supabase](https://supabase.com) 账号并创建新项目
2. 在 SQL Editor 中执行以下建表语句：

```sql
CREATE TABLE IF NOT EXISTS user_data (
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE PRIMARY KEY,
  favorites JSONB DEFAULT '[]'::jsonb,
  workout_history JSONB DEFAULT '[]'::jsonb,
  plan_items JSONB DEFAULT '[]'::jsonb,
  profile JSONB,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

ALTER TABLE user_data ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can access own data"
ON user_data FOR ALL
USING (auth.uid() = user_id)
WITH CHECK (auth.uid() = user_id);
```

> **已有表但缺少 profile 列？** 执行以下语句添加：
> ```sql
> ALTER TABLE user_data ADD COLUMN IF NOT EXISTS profile JSONB;
> ```
>
> **已有 RLS 策略但缺少 WITH CHECK？** 执行以下语句更新：
> ```sql
> DROP POLICY IF EXISTS "Users can access own data" ON user_data;
> CREATE POLICY "Users can access own data"
> ON user_data FOR ALL
> USING (auth.uid() = user_id)
> WITH CHECK (auth.uid() = user_id);
> ```

3. 在项目设置 → API 中获取 URL 和 anon key
4. 修改 `config.js` 填入你的配置：

```javascript
window.FITTRACK_CONFIG = {
  supabaseUrl: 'https://你的项目.supabase.co',
  supabaseAnonKey: '你的anon公钥'
};
```

## 防止 Supabase 免费版项目自动暂停

Supabase 免费版项目如果连续 7 天没有任何 API 请求，会被自动暂停。本项目已内置 GitHub Actions 自动保活脚本，配置步骤如下：

### 步骤一：添加 Secret

1. 进入 GitHub 仓库的 **Settings → Secrets and variables → Actions**
2. 点击 **New repository secret**
3. Name 填：`SUPABASE_ANON_KEY`
4. Secret 填你的 Supabase anon key（即 `config.js` 中 `supabaseAnonKey` 的值，格式如 `sb_publishable_xxx`）
5. 点击 **Add secret**

### 步骤二：启用 Actions

1. 进入 GitHub 仓库的 **Actions** 标签页
2. 如果看到提示，点击 **I understand my workflows, go ahead and enable them**
3. 工作流 `Keep Supabase Alive` 会自动每 6 小时执行一次

### 步骤三：手动测试

1. 在 **Actions** 页面点击 **Keep Supabase Alive**
2. 点击右侧 **Run workflow** → **Run workflow**
3. 等待执行完成，查看日志确认显示 `✅ Supabase is active`

> **原理**：每 6 小时向 Supabase REST API 发送一次请求，保持项目活跃状态。该请求使用 anon key，受 RLS 策略保护，不会泄露或修改任何数据。

## 功能

- 1324+ 健身动作库（含动图演示）
- 自定义训练计划
- 训练计时器（倒计时 + 闹钟提醒）
- 训练记录与历史
- 收藏管理
- 中英双语
- 账号注册登录（Supabase）
- 数据云端同步
