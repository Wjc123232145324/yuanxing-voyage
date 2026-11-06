# 远星航程 · Long Voyage Log

一个以**星际飞船远航**为主题的专注计时器。打开一次航段、把注意力留给当下；中断不再叫「放弃」，而是「紧急返航」。

单文件 HTML，双击即可在浏览器打开；不登录也能离线使用。

**在线体验**：<https://aka.doubaocdn.com/s/UwCoa661Zk>

![航行台](https://aka.doubaocdn.com/s/h7bYdMpnZV)

## 与「种树」类产品的区别

| | 常见专注应用 | 远星航程 |
| --- | --- | --- |
| 专注过程 | 长出一棵树 | 飞行一段航程 |
| 计时单位 | 分钟 | 光年（1 分钟 = 1 光年） |
| 中断 | 树枯死 | 紧急返航，该段不计里程，但仍计入专注日志 |
| 长期成长 | 一片森林 | 8 个星区按累计光年解锁 |

## 功能

- **航行台**：舰外实景主仪表（Canvas 逐帧绘制）——探针舰、三层视差星野、行星大气边光与远月；HUD 叠加大号倒计时、状态灯、光年进度条。
- **计时状态**：STANDBY / CRUISING / HOLD，支持暂停与继续；严格模式下切出页面即判定紧急返航。
- **星区解锁**：01 近地轨道 0 ly → 02 月影带 120 → 03 静默带 320 → 04 灰烬星云 640 → 05 双星岔口 1000 → 06 潮汐环 1500 → 07 远星门 2200 → 08 无光海域 3200。
- **任务清单**：为待办预约航段，按段数排期，进度显示「已完成 N 段」。
- **航行日志**：按日聚合的航段记录，区分抵达 / 主动返航 / 紧急返航。
- **统计**：日分布、时段分布与完成率图表（ECharts，离线自动降级为空态提示）。
- **成就**：12 枚徽章——首次抵达、连续天数、单段深度、累计十小时等。
- **设置**：航段时长、每日目标、严格模式、环境声、音量与实景开关。
- **动效**：统一缓动令牌（expo 收柬 / 弹簧回弹），主控键有键程与掠过光带，环境声分段用滑动指示块，`prefers-reduced-motion` 下全部降级为静态终态。
- **可取消实景**：仪表右上「简洁」可收起动态画面，换成网格读数面板；选择会被记住。

| 巡航中 | 移动端 |
| --- | --- |
| ![巡航](https://aka.doubaocdn.com/s/xcTuu24y4A) | ![移动端](https://aka.doubaocdn.com/s/66sR6PHbiV) |

## 云端账号（可选）

不登录也能用，数据只存在本机浏览器。想要**邮箱密码登录 + 换设备同步**，用你自己的 Supabase 项目接管存储（本仓库不提供、不托管任何服务端）：

1. 在 [supabase.com](https://supabase.com) 建一个免费项目，拿到 **Project URL** 与 **anon public key**（Project Settings → API）。
2. 打开 SQL Editor，执行：

```sql
create table if not exists public.voyages (
  user_id uuid primary key references auth.users(id) on delete cascade,
  data jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);

alter table public.voyages enable row level security;

create policy voyage_read   on public.voyages for select using (auth.uid() = user_id);
create policy voyage_insert on public.voyages for insert with check (auth.uid() = user_id);
create policy voyage_update on public.voyages for update using (auth.uid() = user_id) with check (auth.uid() = user_id);
```

3. 打开应用 → 设置 → 云端账号，填入 URL 与 anon key → 连接并保存 → 注册或登录。

说明：

- 密码由 Supabase Auth 校验，应用只存登录令牌，不存明文密码；anon key 本来就是公开密钥，权限由上面的 RLS 策略限定为「只能读写自己的行」。
- 自用可以在 Authentication → Providers → Email 里关掉 **Confirm email**，注册后免邮箱确认直接登录。
- 登录后本地写入会在 3 秒内自动上传；两端都有数据时不会默默覆盖，让你选「上传本地」或「从云端恢复」。
- 计时进行中不执行云端覆盖，避免把正在跑的航段弄丢。

## 打开方式

```bash
open 远星航程.html        # macOS
xdg-open 远星航程.html    # Linux
```

本地数据存在浏览器 `localStorage`：航程在 `yuanxing-voyage-v1`，云端项目配置在 `yx-cloud-config`。

## 技术要点

- 单文件：CSS / JS 全部内联，9 张星区美术图以 base64 内联，断网可用；云端模块（supabase-js）只在用到时懒加载。
- 原生 JS：无框架、无构建步骤；状态存于 `S = { settings, missions, sessions, active }`。
- 登录走后端托管服务（Supabase Auth + Postgres + RLS），仓库本身仍是纯静态页面，可直接丢到任意静态托管。
- 双端自检：桌面 1440×1000 与移动 390×844 渲染无报错、无横向溢出。

## 文件

| 文件 | 说明 |
| --- | --- |
| `远星航程.html` | 应用本体，单文件（约 309 KB，含内联美术素材） |
| `LICENSE` | MIT |

## License

MIT © 2026 Wjc123232145324
