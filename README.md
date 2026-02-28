# 表情包搜索插件（内置多接口轮询）

根据关键词搜索表情包图片，内置多条 API 接口并自动轮询，确保在某个接口搜索不到或结果为空时继续尝试下一个接口。

## 功能特性
- 内置 3 个接口并自动轮询：apihzbqb.php、apihzbqbbaidu.php、apihzbqbsougou.php
- 统一参数构造，自动限制 `limit≤20`，规范 `page≥1`
- 兼容不同返回结构（含无 count/maxpage 的接口）
- 提供图片下载工具方法

## 快速开始
- 代码入口位于 [init.py](file:///c:/Users/ASUS/Documents/trae_projects/cs/init.py)
- 搜索示例：

```python
# 在多模态代理中使用
result_text = await search_emoji(ctx, "伤心")
print(result_text)

# 下载图片
image_bytes = await get_emoji_image(ctx, "https://example.com/xxx.jpg")
```

关键方法：
- 轮询与参数构造：[fetch_emoji_images](file:///c:/Users/ASUS/Documents/trae_projects/cs/init.py#L48-L112)
- 结果格式化兼容：[format_result](file:///c:/Users/ASUS/Documents/trae_projects/cs/init.py#L114-L147)

## 配置说明
配置类：`EmojiSearchConfig`

- `USER_ID`：用户中心数字 ID（必填）。获取地址：<https://www.apihz.cn/>
- `USER_KEY`：用户中心通讯秘钥（必填）。获取地址：<https://www.apihz.cn/>
- `EXTRA_KEYWORD`：额外关键词（可选），会自动拼接到搜索词
- `TIMEOUT`：请求超时时间（秒）

说明：已移除 `API_URL` 配置项，接口地址改为内置并轮询调用。

## 内置接口与参数
接口轮询顺序：
1. `https://cn.apihz.cn/api/img/apihzbqb.php`（新参数版）
   - 参数：`id`、`key`、`type=2`、`words`（URL 编码，最长约 10 个汉字）、`page`、`limit≤20`
2. `https://cn.apihz.cn/api/img/apihzbqbbaidu.php`（百度版）
   - 参数：`id`、`key`、`words`（URL 编码）、`page`、`limit≤20`
3. `https://cn.apihz.cn/api/img/apihzbqbsougou.php`（搜狗版）
   - 参数：`id`、`key`、`words`（URL 编码，最长约 100 个汉字）、`page`

所有接口统一传递 `words`（搜索词与 `EXTRA_KEYWORD` 拼接后 URL 编码），`page` 规范为≥1。

## 返回参数说明
- `code`：状态码（200 成功，400 错误）
- `msg`：错误提示（当 `code=400` 时返回）
- `res`：表情包图片地址结果集
- `page`：当前页码（部分接口返回）
- `maxpage`、`count`：仅在支持分页统计的接口返回

展示示例：
```
找到<count或len(res)>个表情包，当前第<page>/<maxpage或当前页>页
随机选择一个: <某个图片URL>
```

错误返回示例（当缺少必要参数时）：
```json
{
  "code": 400,
  "msg": "id参数不能为空，请前往接口盒子官网获取。",
  "官网": "www.apihz.cn"
}
```

## 常见问题
- 400 错误：检查是否正确传入 `id`、`key`、`type`（仅限 apihzbqb.php）以及关键词长度与编码。
- 返回为空：当 `res` 为空会继续轮询下一个接口；若全部为空则提示“没有找到匹配的表情包”，建议更换关键词或调整页码。
- 接口限制：`limit` 仅在支持该参数的接口提交；搜狗接口不使用 `limit`。

## 作者与来源
- 作者：XGGM
- 项目地址：<https://github.com/XG2020/emo_plugin>
