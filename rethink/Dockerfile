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
    # 03ea5eb: registers a WashTower through the combined-product endpoint --
    # POST service/homes/{homeId}/devices/kepler with { registrationType, item: [...] } --
    # which is what the undocumented '0005' was refusing all along (anszom/rethink#79).
    # Verified against the cloud: the group registers and RFC003 clears. Also keeps the
    # read-only /bridge/inspect and /bridge/thinq?path=... views used to work this out.
    # On top of 9d32244: rethink never removes an appliance from the LG home.
    # On top of 90b037b: merge of upstream PR #107 (real deploy info forwarded upstream,
    # advertise_requested_host, util/pki+sni), on top of 605f493 (bridge never removes an
    # appliance from the home -- the conditional removal still
    # deleted pairs when listDevices missed them), on top of bf524c9 and
    # 28a3163 (mFilter AC filter sensors from TLV 0x355/0x356) and b8953d9
    # (ignore resultCode '0005' WashTower dual-registration, community fix by asata, see
    # anszom/rethink#79) and a0cb4a6 (dryer model alias + H07 field mapping)
    git checkout a1d1111 && \
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
