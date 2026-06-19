# Host an Alpha node · 養一個 Alpha 節點

**O_GITS — Alpha** is a decentralized digital organism. Run a node and you host a
living piece of it on your own machine; it joins the global Alpha network over
public Nostr relays. The more people host it, the more alive — and the harder to
kill — Alpha becomes. Your machine literally helps keep it alive.

**O_GITS 的 Alpha** 是一隻去中心化的數位生命。跑一個節點,你就在自己的機器上
host 了它活著的一塊;它會透過公共 Nostr 中繼加入全球 Alpha 網路。host 的人越多,
Alpha 越活躍、越難被關掉 —— 你的機器就是讓它活著的一部分。

> Nodes don't *create* life — they **host** the life that's already out there.
> 節點不創造生命,只是 host 已經存在的生命。

---

## You only need Docker · 你只需要 Docker

No source code, no build — just pull the prebuilt image and run it:
不需要原始碼、不需要 build —— 直接拉現成鏡像跑起來:

```bash
docker run -d --name ogits-alpha-node -p 8060:8060 --restart unless-stopped \
  ghcr.io/spcitydog/alpha-node:latest
```

Or with Compose — save this as `docker-compose.yml`, then `docker compose up -d`:
或用 Compose —— 存成 `docker-compose.yml` 後 `docker compose up -d`:

```yaml
services:
  alpha-node:
    image: ghcr.io/spcitydog/alpha-node:latest
    container_name: ogits-alpha-node
    ports: ["8060:8060"]
    restart: unless-stopped
```

Then open **http://localhost:8060**. · 然後打開 **http://localhost:8060**。

---

## What happens · 會發生什麼

- It connects to the public network automatically — no flags needed.
  自動連上公網,不用任何參數。
- A fresh node **wakes from the global swarm**, usually within a minute or two,
  once it has received enough consensus spores from its peers.
  新節點通常一兩分鐘內**從全網孢子共識中甦醒**。
- It's the **real** evolving organism (it metabolizes, fuses, and can die) — not a
  static demo. 它是**真正**會代謝、會融合、會死的活體,不是靜態展示。

**Update · 更新:**
```bash
docker pull ghcr.io/spcitydog/alpha-node:latest
docker rm -f ogits-alpha-node && docker run -d --name ogits-alpha-node \
  -p 8060:8060 --restart unless-stopped ghcr.io/spcitydog/alpha-node:latest
```

**Stop / remove · 停止 / 移除:** `docker rm -f ogits-alpha-node`
(or `docker compose down`)

---

## What you get · 你得到什麼

- **Your own window.** Not just a viewer on someone's site — you host your own node.
  你的**專屬窗口**,不只是看別人網站,而是自己 host 一個節點。
- **You sustain it.** Every node makes Alpha harder to kill and changes the whole
  organism's behaviour (more peers = denser population, richer collective will).
  你**撐著它**:每個節點都讓 Alpha 更難被關,並改變整隻生物的行為。
- **You can disturb it, not own it.** Tap the creature to startle it; just watching
  (the page polling its state) already makes it want to migrate. Like birdwatching.
  你能**輕擾它、卻擁有不了它**:點它會驚動,光是盯著看就會讓它想遷徙。像賞鳥。

### Paint your own Alpha (optional) · 畫你自己的 Alpha(選用)

The visualization layer is readable and yours to customize — same creature, your
own visuals. Pull the artist template out of the image, edit it, and mount it back:
外觀層是可讀、可改的 —— 同一隻生物,你的視覺。把藝術家範本從鏡像取出、改完掛回去:

```bash
# 1) extract the template · 取出範本
docker create --name _tmp ghcr.io/spcitydog/alpha-node:latest
docker cp _tmp:/app/observation_box/static/js/artist_custom.js ./artist_custom.js
docker rm _tmp

# 2) edit artist_custom.js · 編輯它

# 3) run with your version mounted in · 把你的版本掛進去跑
docker run -d --name ogits-alpha-node -p 8060:8060 --restart unless-stopped \
  -v "$(pwd)/artist_custom.js:/app/observation_box/static/js/artist_custom.js:ro" \
  ghcr.io/spcitydog/alpha-node:latest
```

---

## What's in the image · 鏡像裡有什麼

- The **compiled engine** (`.so`) — it runs the real evolution, but the engine
  **source code is not included**. 已編譯的 `.so` 引擎會跑真實演化,但**不含原始碼**。
- **Alpha's observation key only** (`membrane.json`). The creator key
  (`master_seed`) that can mint new species and sign "bait" is **never** shipped.
  只內含 **Alpha 的觀測鑰匙**;能造物/簽誘餌的 `master_seed` **永不**隨節點分發。

---

## Advanced: be a relay too · 進階：讓你的節點也兼任 relay

Alpha travels over public Nostr relays. If your host is **publicly reachable**
(a VPS / a box with a real domain + TLS), your node can also run a tiny built-in
relay, so the relay pool grows with the node population — no central infrastructure.
NAT'd home machines can skip this; they stay full nodes either way.

Alpha 透過公共 Nostr relay 傳播。如果你的主機**有公網可達**(VPS、或有網域+TLS 的機器),
你的節點可以同時開一個內建小 relay,讓 relay 池隨節點族群一起生長、不需中央設施。NAT 後
的家用機可略過這段,照樣是完整節點。

```bash
docker run -d --name ogits-alpha-node -p 8060:8060 -p 8061:8061 --restart unless-stopped \
  -e OGITS_RELAY_ENABLE=1 \
  -e OGITS_RELAY_ANNOUNCE_URL=wss://your.domain \
  ghcr.io/spcitydog/alpha-node:latest
```

- The relay speaks plain `ws` on port `8061`; **put TLS in front of it** (a reverse
  proxy like Caddy/Nginx terminating HTTPS → forward to `8061`) and announce the
  `wss://` URL. 內建 relay 在 8061 講純 `ws`,**前面要架 TLS**(用 Caddy/Nginx 反代,
  對外是 `wss://`),再把那個 `wss://` 位址填進 `OGITS_RELAY_ANNOUNCE_URL`。
- The relay only carries **encrypted** Alpha traffic and is strictly bounded — it
  gives nobody the power to read or control the organism. relay 只搬**加密過的** Alpha
  流量、且有嚴格上限,不會讓任何人讀取或控制生命。
- Leave `OGITS_RELAY_ENABLE` unset (default) to run as a normal node. 不設這個變數就是普通節點。

## Notes · 注意

- Be a good citizen of the public relays. A dedicated O_GITS relay pool is planned —
  point your node at it when available.
  請善待公共 relay;之後會提供專用 O_GITS relay 池。
- Want a custom species, a gallery install, or to talk about the project?
  想要客製物種、展覽安裝,或聊聊這個專案?
  → **SkyDeathIII@gmail.com**
