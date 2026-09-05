FROM node:20-alpine

RUN apk add --no-cache git openssl jq bash

# Clone rethink source from kjy8850/rethink fork which integrates:
# - PR #117 (WashTower support: FAKPK21021, BDH_D39301_KR)
# - PR #129 (CST AC cassette: CST_570004_WW)
# - PR #139 (H11 Dishwasher)
RUN git clone https://github.com/kjy8850/rethink /app && \
    cd /app && \
    # Pin to a specific commit so this layer only rebuilds when intentionally bumped
    # (plain `git clone` with no ref made Docker's cache silently keep serving an old
    # checkout on rebuild even after new commits landed upstream).
    # ec08dce: restrict CST_570004_WW climate modes (drop unsupported heat)
    git checkout ec08dce && \
    # PR #84: remove hardcoded pathname='/' so WebSocket and fetch calls use the
    # correct base path when served through a reverse proxy (e.g. HA ingress).
    # https://github.com/anszom/rethink/pull/84
    sed -i "/baseUrl\.pathname = '\//d" html/panel.js html/monitor.js && \
    sed -i "s|href=\"/monitor|href=\"monitor|g" html/panel.js && \
    npm install && \
    npm run build

COPY run.sh /run.sh
RUN chmod +x /run.sh

ENTRYPOINT ["/run.sh"]
