# PanSou 部署说明

## 项目简介

PanSou是一个高性能的网盘资源搜索API服务，支持TG搜索和自定义插件搜索。系统设计以性能和可扩展性为核心，支持并发搜索、结果智能排序和网盘类型分类。

## 优化内容

本次优化实现了搜索到数据后，每个频道只保留最新的最高清资源的功能。

1. **清晰度提取功能**：从标题中提取清晰度信息，支持8K、4K、2160P、1080P等多种清晰度格式
2. **频道过滤逻辑**：每个频道只保留最新的最高清资源
3. **搜索结果优化**：减少冗余信息，提高搜索效率

## 部署方式

### 1. 使用Docker部署（推荐）

#### 直接使用Docker命令

```bash
docker run -d --name pansou -p 9999:8888 ghcr.io/easy1984/pansou:latest
```

#### 使用Docker Compose

```yaml
version: '3.8'

services:
  pansou:
    image: ghcr.io/Gg8-66/pansou:latest
    container_name: pansou
    restart: unless-stopped
    ports:
      - "8888:8888"
    environment:
      - PORT=8888
      - CHANNELS=tgsearchers4,Aliyun_4K_Movies,bdbdndn11,yunpanx,bsbdbfjfjff,yp123pan,sbsbsnsqq,yunpanxunlei,tianyifc,BaiduCloudDisk,txtyzy,peccxinpd,gotopan,PanjClub,kkxlzy,baicaoZY,MCPH01,MCPH02,MCPH03,bdwpzhpd,ysxb48,jdjdn1111,yggpan,MCPH086,zaihuayun,Q66Share,ucwpzy,shareAliyun,alyp_1,dianyingshare,Quark_Movies,XiangxiuNBB,ydypzyfx,ucquark,xx123pan,yingshifenxiang123,zyfb123,tyypzhpd,tianyirigeng,cloudtianyi,hdhhd21,Lsp115,oneonefivewpfx,qixingzhenren,taoxgzy,Channel_Shares_115,tyysypzypd,vip115hot,wp123zy,yunpan139,yunpan189,yunpanuc,yydf_hzl,leoziyuan,pikpakpan,Q_dongman,yoyokuakeduanju,TG654TG,WFYSFX02,QukanMovie,yeqingjie_GJG666,movielover8888_film3,Baidu_netdisk,D_wusun,FLMdongtianfudi,KaiPanshare,QQZYDAPP,rjyxfx,PikPak_Share_Channel,btzhi,newproductsourcing,cctv1211,duan_ju,QuarkFree,yunpanNB,kkdj001,xxzlzn,pxyunpanxunlei,jxwpzy,kuakedongman,liangxingzhinan,xiangnikanj,solidsexydoll,guoman4K,zdqxm,kduanju,cilidianying,CBduanju,SharePanFilms,dzsgx,BooksRealm,Oscar_4Kmovies,douerpan,baidu_yppan,Q_jilupian,Netdisk_Movies,yunpanquark,ammmziyuan,ciliziyuanku,cili8888,jzmm_123pan
      - ENABLED_PLUGINS=labi,zhizhen,shandian,duoduo,muou,wanou,hunhepan,jikepan,panwiki,pansearch,panta,qupansou,hdr4k,pan666,susu,thepiratebay,xuexizhinan,panyq,ouge,huban,cyg,erxiao,miaoso,fox4k,pianku,clmao,wuji,cldi,xiaozhang,libvio,leijing,xb6v,xys,ddys,hdmoli,yuhuage,u3c3,javdb,clxiong,jutoushe,sdso,xiaoji,xdyh,haisou,bixin,djgou,nyaa,xinjuc,aikanzy,qupanshe,xdpan,discourse,yunsou,qqpd,ahhhhfs,nsgame,gying,quark4k,quarksoo,sousou,ash
      - CACHE_ENABLED=true
      - CACHE_PATH=/app/cache
      - CACHE_MAX_SIZE=100
      - CACHE_TTL=60
      - ASYNC_PLUGIN_ENABLED=true
      - ASYNC_RESPONSE_TIMEOUT=4
      - ASYNC_MAX_BACKGROUND_WORKERS=20
      - ASYNC_MAX_BACKGROUND_TASKS=100
      - ASYNC_CACHE_TTL_HOURS=1
    volumes:
      - pansou-cache:/app/cache
    networks:
      - pansou-network
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:8888/api/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s

volumes:
  pansou-cache:
    name: pansou-cache

networks:
  pansou-network:
    name: pansou-network
```

### 2. 从源码部署

#### 环境要求

- Go 1.18+
- 可选：SOCKS5代理（用于访问受限地区的Telegram站点）

#### 部署步骤

1. **克隆仓库**

```bash
git clone https://github.com/Gg8-66/pansou.git
cd pansou
```

2. **构建程序**

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags="-s -w -extldflags '-static'" -o pansou .
```

3. **运行程序**

```bash
./pansou
```

## API使用

### 搜索API

**接口地址**：`/api/search`
**请求方法**：`POST` 或 `GET`

**POST请求示例**：

```bash
curl -X POST http://localhost:8888/api/search \
  -H "Content-Type: application/json" \
  -d '{"kw":"速度与激情"}'
```

**GET请求示例**：

```bash
curl "http://localhost:8888/api/search?kw=速度与激情&res=merge&src=tg"
```

## 配置说明

### 基础配置

| 环境变量 | 描述 | 默认值 |
|----------|------|--------|
| **PORT** | 服务端口 | `8888` |
| **PROXY** | SOCKS5代理 | 无 |
| **CHANNELS** | 默认搜索的TG频道 | `tgsearchers3` |
| **ENABLED_PLUGINS** | 指定启用插件 | 无 |

### 高级配置

| 环境变量 | 描述 | 默认值 |
|----------|------|--------|
| CONCURRENCY | 并发搜索数 | 自动计算 |
| CACHE_TTL | 缓存有效期（分钟） | `60` |
| CACHE_MAX_SIZE | 最大缓存大小(MB) | `100` |
| PLUGIN_TIMEOUT | 插件超时时间(秒) | `30` |
| ASYNC_RESPONSE_TIMEOUT | 快速响应超时(秒) | `4` |

## 健康检查

**接口地址**：`/api/health`
**请求方法**：`GET`

```bash
curl http://localhost:8888/api/health
```

## 注意事项

1. 确保Docker环境已正确安装和配置
2. 如果需要访问Telegram，可能需要配置代理
3. 首次运行时，程序会自动初始化缓存目录
4. 建议定期清理缓存，以避免占用过多磁盘空间
5. 搜索结果会按照插件等级、时间新鲜度和优先关键词进行排序
6. 每个频道只会返回一个结果，即该频道最新的最高清资源

## 许可证

本项目采用 MIT 许可证。详情请见 [LICENSE](LICENSE) 文件。
