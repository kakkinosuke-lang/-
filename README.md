<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>日本経済シミュレーター v4</title>
<style>
*{margin:0;padding:0;box-sizing:border-box;}
body{background:#0f0f1e;color:#e0e0e0;font-family:'Segoe UI','Meiryo',sans-serif;overflow:hidden;user-select:none;font-size:16px;}

/* TOP BAR */
#topBar{position:fixed;top:0;left:0;right:0;height:66px;background:linear-gradient(90deg,rgba(7,18,42,.96),rgba(15,34,72,.94));backdrop-filter:blur(12px);display:flex;align-items:center;padding:0 10px;z-index:100;border-bottom:2px solid #e94560;gap:4px;overflow-x:auto;box-shadow:0 8px 22px rgba(0,0,0,.28);}
#topBar .stat{display:flex;flex-direction:column;align-items:center;padding:4px 8px;border-right:1px solid rgba(255,255,255,.1);min-width:96px;}
#topBar .stat.clickable{cursor:pointer;transition:background .12s ease;}
#topBar .stat.clickable:hover{background:rgba(255,255,255,.06);}
#topBar .stat .lbl{font-size:10px;color:#8894b0;text-transform:uppercase;white-space:nowrap;}
#topBar .stat .val{font-size:17px;font-weight:bold;white-space:nowrap;}
#topBar .stat .rate{font-size:10px;}
.rate-pos{color:#4ecca3;}.rate-neg{color:#e94560;}
#weekDisplay{font-size:18px;color:#f39c12;font-weight:bold;white-space:nowrap;margin-left:auto;padding:0 12px;}

/* PRICE BAR */
#priceBar{position:fixed;top:66px;left:0;right:0;height:42px;background:linear-gradient(90deg,rgba(7,17,39,.94),rgba(10,28,58,.9));backdrop-filter:blur(10px);display:flex;align-items:center;padding:0 10px;z-index:99;border-bottom:1px solid #1a2a50;gap:14px;overflow-x:auto;}
.price-item{display:flex;align-items:center;gap:5px;font-size:13px;white-space:nowrap;cursor:pointer;padding:4px 8px;border-radius:8px;transition:.12s;}
.price-item:hover{background:rgba(255,255,255,.05);}
.price-item .dot{width:8px;height:8px;border-radius:50%;flex-shrink:0;}
.price-up{color:#4ecca3;}.price-down{color:#e94560;}

/* GAME AREA */
#gameArea{position:fixed;top:108px;left:0;right:clamp(400px,30vw,510px);bottom:64px;overflow:hidden;cursor:grab;transition:right .12s ease, box-shadow .12s ease, top .12s ease, bottom .12s ease;background:
  radial-gradient(circle at 18% 20%, rgba(255,255,255,.04), transparent 26%),
  linear-gradient(180deg,#020403,#030605);}
#gameArea:active{cursor:grabbing;}
canvas#mapCanvas{display:block;width:100%;height:100%;position:relative;z-index:1;}
#gameArea::before{content:"";position:absolute;inset:0;pointer-events:none;z-index:0;background:
  linear-gradient(rgba(255,255,255,.012) 1px, transparent 1px),
  linear-gradient(90deg, rgba(255,255,255,.012) 1px, transparent 1px);
  background-size:72px 72px,72px 72px;
  opacity:.05;}
body.focus-map #gameArea{right:clamp(400px,30vw,510px);box-shadow:inset -10px 0 25px rgba(0,0,0,.22);}
body.focus-panel #gameArea{right:clamp(400px,30vw,510px);}
body.mode-world-map #gameArea{background:radial-gradient(circle at 50% 35%, rgba(90,255,149,.06), transparent 34%),linear-gradient(180deg,#010302,#020503);}
body.mode-world-map #gameArea::before{opacity:.02;background-size:96px 96px,96px 96px;}
body.mode-japan-map #gameArea{background:radial-gradient(circle at 50% 28%, rgba(193,230,201,.055), transparent 32%),linear-gradient(180deg,#010302,#020403);}
body.mode-space-map #gameArea{background:radial-gradient(circle at 50% 28%, rgba(105,171,255,.07), transparent 34%),linear-gradient(180deg,#030712,#060b18);}

/* RIGHT PANEL */
#rightPanel{position:fixed;top:108px;right:0;width:clamp(400px,30vw,510px);bottom:64px;background:linear-gradient(180deg,rgba(11,18,39,.98),rgba(9,16,33,.97));border-left:1px solid rgba(78,129,212,.18);display:flex;flex-direction:column;transition:width .12s ease, box-shadow .12s ease, top .12s ease, bottom .12s ease;box-shadow:-16px 0 30px rgba(0,0,0,.18);}
#panelTabs{display:flex;flex-wrap:wrap;gap:7px;padding:10px;background:linear-gradient(180deg,rgba(10,18,38,.95),rgba(8,15,31,.9));border-bottom:1px solid rgba(78,129,212,.14);position:sticky;top:0;z-index:6;backdrop-filter:blur(10px);}
#panelTabs .btn{font-size:14px;padding:9px 13px;border-radius:999px;}
#panelContent{flex:1;overflow-y:auto;padding:14px 14px 22px;font-size:15px;background:
  radial-gradient(circle at 50% 0%, rgba(52,152,219,.06), transparent 28%),
  linear-gradient(180deg,rgba(255,255,255,.01),transparent 20%);}
body.focus-map #rightPanel{width:clamp(400px,30vw,510px);}
body.focus-panel #rightPanel{width:clamp(400px,30vw,510px);box-shadow:-10px 0 25px rgba(0,0,0,.28);}
body.mode-world-map #rightPanel{background:linear-gradient(180deg,rgba(7,17,10,.98),rgba(5,12,8,.97));border-left:1px solid rgba(92,241,149,.18);}
body.mode-world-map #panelTabs{background:linear-gradient(180deg,rgba(9,22,12,.95),rgba(7,16,9,.92));border-bottom:1px solid rgba(92,241,149,.12);}
body.mode-world-map #panelContent{background:radial-gradient(circle at 50% 0%, rgba(96,255,156,.07), transparent 30%),linear-gradient(180deg,rgba(255,255,255,.008),transparent 22%);}
body.mode-japan-map #rightPanel{background:linear-gradient(180deg,rgba(12,22,18,.98),rgba(9,16,14,.97));border-left:1px solid rgba(189,231,199,.16);}
body.mode-japan-map #panelTabs{background:linear-gradient(180deg,rgba(15,29,22,.95),rgba(11,21,17,.92));border-bottom:1px solid rgba(189,231,199,.11);}
body.mode-japan-map #panelContent{background:radial-gradient(circle at 50% 0%, rgba(190,229,199,.06), transparent 28%),linear-gradient(180deg,rgba(255,255,255,.008),transparent 22%);}
body.mode-space-map #rightPanel{background:linear-gradient(180deg,rgba(10,16,33,.98),rgba(8,13,27,.97));border-left:1px solid rgba(114,175,255,.16);}
body.mode-space-map #panelTabs{background:linear-gradient(180deg,rgba(12,20,42,.95),rgba(8,16,31,.92));border-bottom:1px solid rgba(114,175,255,.11);}
body.mode-space-map #panelContent{background:radial-gradient(circle at 50% 0%, rgba(114,175,255,.07), transparent 30%),linear-gradient(180deg,rgba(255,255,255,.008),transparent 22%);}

/* BOTTOM BAR */
#bottomBar{position:fixed;bottom:0;left:0;right:0;height:64px;background:linear-gradient(90deg,rgba(9,22,51,.96),rgba(17,36,78,.94));backdrop-filter:blur(12px);display:flex;align-items:center;padding:0 10px;z-index:100;border-top:2px solid #e94560;gap:9px;overflow-x:auto;box-shadow:0 -8px 22px rgba(0,0,0,.24);}
#speedCtrl{display:flex;align-items:center;gap:3px;margin-left:auto;}
#speedCtrl .speed-divider{width:1px;height:24px;background:rgba(255,255,255,.12);margin:0 4px;border-radius:999px;}

/* BUTTONS */
.btn{padding:8px 14px;border:1px solid #e94560;background:rgba(255,255,255,.02);color:#ddd;border-radius:10px;cursor:pointer;font-size:15px;transition:all .12s;white-space:nowrap;box-shadow:inset 0 1px 0 rgba(255,255,255,.04);}
.btn:hover{background:#e94560;color:#fff;}
.btn.active{background:#e94560;color:#fff;}
.btn:disabled{opacity:.4;cursor:default;}.btn:disabled:hover{background:transparent;color:#ddd;}
.btn-green{border-color:#4ecca3;}.btn-green:hover,.btn-green.active{background:#4ecca3;color:#000;}
.btn-blue{border-color:#3498db;}.btn-blue:hover,.btn-blue.active{background:#3498db;color:#fff;}
.btn-yellow{border-color:#f39c12;}.btn-yellow:hover,.btn-yellow.active{background:#f39c12;color:#000;}
.btn-purple{border-color:#9b59b6;}.btn-purple:hover,.btn-purple.active{background:#9b59b6;color:#fff;}
.btn-sm{font-size:13px;padding:6px 10px;}

/* SECTIONS */
.section{margin-bottom:10px;background:rgba(255,255,255,.03);border-radius:8px;padding:10px;border:1px solid rgba(255,255,255,.06);}
.section h3{font-size:18px;color:#e94560;margin-bottom:8px;border-bottom:1px solid rgba(255,255,255,.06);padding-bottom:5px;}
body.mode-japan-map .section{background:linear-gradient(180deg,rgba(194,231,201,.06),rgba(255,255,255,.025));border-color:rgba(194,231,201,.12);}
body.mode-japan-map .company,
body.mode-japan-map .research-card,
body.mode-japan-map .goal-card,
body.mode-japan-map .macro-card{background:linear-gradient(180deg,rgba(194,231,201,.05),rgba(0,0,0,.22));border-color:rgba(194,231,201,.1);}
body.mode-world-map .section{background:linear-gradient(180deg,rgba(96,255,156,.06),rgba(255,255,255,.022));border-color:rgba(96,255,156,.12);}
body.mode-world-map .company,
body.mode-world-map .research-card,
body.mode-world-map .goal-card,
body.mode-world-map .macro-card{background:linear-gradient(180deg,rgba(96,255,156,.05),rgba(0,0,0,.2));border-color:rgba(96,255,156,.1);}
body.mode-space-map .section{background:linear-gradient(180deg,rgba(114,175,255,.065),rgba(255,255,255,.02));border-color:rgba(114,175,255,.12);}
body.mode-space-map .company,
body.mode-space-map .research-card,
body.mode-space-map .goal-card,
body.mode-space-map .macro-card{background:linear-gradient(180deg,rgba(114,175,255,.055),rgba(0,0,0,.2));border-color:rgba(114,175,255,.11);}

/* COMPANY */
.company{padding:11px 11px;margin:6px 0;border-radius:8px;background:rgba(0,0,0,.2);cursor:pointer;transition:all .12s;border-left:4px solid #555;display:flex;align-items:center;justify-content:space-between;gap:8px;font-size:15px;}
.company:hover{background:rgba(255,255,255,.05);}
.company .c-left{display:flex;align-items:center;gap:4px;min-width:0;}
.company .c-name{font-size:15px;font-weight:bold;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;}
.company .c-right{display:flex;align-items:center;gap:6px;flex-shrink:0;}
.company .c-price{font-size:15px;font-weight:bold;}
.company .c-change{font-size:13px;}

/* RESOURCE BAR */
.resource-bar{display:flex;align-items:center;margin:6px 0;font-size:14px;padding:4px 0;gap:6px;}
.resource-bar .r-name{width:120px;color:#aaa;flex-shrink:0;}
.resource-bar .r-bar{flex:1;height:10px;background:rgba(0,0,0,.3);border-radius:999px;margin:0 4px;overflow:hidden;}
.resource-bar .r-fill{height:100%;border-radius:3px;transition:width .3s;}
.resource-bar .r-val{width:110px;text-align:right;font-size:11px;flex-shrink:0;}
.resource-zero .r-name{color:#e94560;}

/* TOOLTIP */
#tooltip{position:fixed;background:#151a30;border:1px solid #e94560;border-radius:8px;padding:9px 10px;font-size:12px;z-index:500;pointer-events:none;display:none;max-width:320px;box-shadow:0 4px 15px rgba(0,0,0,.5);}

/* EVENT LOG */
#eventLog{position:relative;top:0;left:0;width:100%;max-height:none;overflow:visible;z-index:auto;pointer-events:auto;}
.floating-dock{position:fixed;z-index:72;width:min(360px,42vw);border-radius:18px;background:rgba(9,16,30,.74);border:1px solid rgba(255,255,255,.08);backdrop-filter:blur(14px);box-shadow:0 18px 36px rgba(0,0,0,.3);overflow:hidden;transition:box-shadow .16s ease, transform .18s ease;}
.floating-dock:hover{box-shadow:0 22px 44px rgba(0,0,0,.34);}
.floating-dock-header{display:flex;align-items:center;justify-content:space-between;gap:10px;padding:10px 12px;background:linear-gradient(180deg,rgba(255,255,255,.05),rgba(255,255,255,.02));cursor:grab;user-select:none;}
.floating-dock-title{font-size:12px;font-weight:800;color:#eff7ff;letter-spacing:.04em;}
.floating-dock-actions{display:flex;gap:6px;align-items:center;}
.floating-dock-handle,.floating-dock-collapse{width:28px;height:28px;border-radius:999px;border:1px solid rgba(255,255,255,.14);background:rgba(255,255,255,.04);color:#eef7ff;display:inline-flex;align-items:center;justify-content:center;font-size:13px;font-weight:800;cursor:pointer;}
.floating-dock-body{max-height:260px;overflow:auto;padding:4px 8px 10px;transition:max-height .24s ease, opacity .2s ease, padding .2s ease;opacity:1;}
.floating-dock.collapsed .floating-dock-body{max-height:0;opacity:0;padding-top:0;padding-bottom:0;overflow:hidden;}
.event-msg{background:rgba(15,15,30,.92);border-left:3px solid #f39c12;padding:6px 10px;margin:4px 0;font-size:13px;border-radius:0 6px 6px 0;animation:fadeIn .22s;}
.event-msg.bad{border-left-color:#e94560;}.event-msg.good{border-left-color:#4ecca3;}
@keyframes fadeIn{from{opacity:0;transform:translateX(-15px);}to{opacity:1;transform:translateX(0);}}

#majorEventBanner{position:fixed;top:116px;left:50%;transform:translateX(-50%);min-width:420px;max-width:70vw;background:linear-gradient(90deg,rgba(233,69,96,.95),rgba(243,156,18,.95));color:#fff;padding:10px 16px;border-radius:999px;z-index:140;font-size:16px;font-weight:bold;box-shadow:0 8px 20px rgba(0,0,0,.35);display:none;text-align:center;pointer-events:none;}
#majorEventBanner.show{display:block;animation:bannerPop .35s ease;}
@keyframes bannerPop{from{opacity:0;transform:translateX(-50%) translateY(-12px);}to{opacity:1;transform:translateX(-50%) translateY(0);}}
#cinematicEvent{position:fixed;inset:0;display:none;align-items:center;justify-content:center;pointer-events:none;z-index:980;overflow:hidden;}
#cinematicEvent.show{display:flex;}
#cinematicEventBackdrop{position:absolute;inset:0;background:radial-gradient(circle at center,rgba(255,216,128,.18),rgba(0,0,0,.04) 26%,rgba(0,0,0,.58) 78%);opacity:0;}
#cinematicEvent.show #cinematicEventBackdrop{animation:cinematicBackdrop 3.8s ease forwards;}
#cinematicEventCard{position:relative;min-width:min(720px,82vw);max-width:82vw;padding:34px 40px 38px;border-radius:28px;border:1px solid rgba(255,255,255,.12);background:linear-gradient(180deg,rgba(12,22,44,.92),rgba(8,14,28,.86));box-shadow:0 26px 64px rgba(0,0,0,.46);text-align:center;overflow:hidden;transform:translateY(24px) scale(.92);opacity:0;}
#cinematicEvent.show #cinematicEventCard{animation:cinematicCardIn 4.2s cubic-bezier(.18,.78,.22,1) forwards;}
#cinematicEvent.victory #cinematicEventCard{background:linear-gradient(180deg,rgba(34,26,12,.94),rgba(15,10,4,.9));border-color:rgba(255,220,140,.26);}
#cinematicEvent.defeat #cinematicEventCard{background:linear-gradient(180deg,rgba(42,14,18,.94),rgba(18,7,10,.9));border-color:rgba(255,134,134,.24);}
.cinematic-badge{position:absolute;top:16px;right:18px;padding:7px 12px;border-radius:999px;background:rgba(255,255,255,.08);border:1px solid rgba(255,255,255,.12);font-size:12px;font-weight:800;letter-spacing:.08em;color:#f4f7fb;text-transform:uppercase;}
.cinematic-icon{position:relative;z-index:2;font-size:68px;line-height:1;margin-bottom:12px;filter:drop-shadow(0 8px 18px rgba(0,0,0,.28));}
.cinematic-title{position:relative;z-index:2;font-size:42px;font-weight:900;letter-spacing:.06em;color:#fff;margin-bottom:10px;text-shadow:0 6px 18px rgba(0,0,0,.28);}
.cinematic-subtitle{position:relative;z-index:2;font-size:18px;line-height:1.7;color:rgba(244,247,251,.86);max-width:720px;margin:0 auto;}
.cinematic-smoke{position:absolute;inset:-120px;overflow:visible;}
.cinematic-smoke .smoke-puff{position:absolute;left:50%;top:50%;border-radius:50%;background:radial-gradient(circle,rgba(255,255,255,.42) 0%,rgba(190,190,190,.18) 44%,rgba(60,60,60,0) 78%);filter:blur(5px);mix-blend-mode:screen;opacity:0;transform:translate(-50%,-50%) scale(.26);animation:smokeLift var(--dur,3.2s) ease-out forwards;animation-delay:var(--delay,0s);}
.cinematic-smoke .spark{position:absolute;left:50%;top:50%;width:7px;height:7px;border-radius:50%;background:radial-gradient(circle,rgba(255,248,210,.95) 0%,rgba(255,206,92,.32) 64%,rgba(255,206,92,0) 100%);opacity:0;transform:translate(-50%,-50%);animation:sparkBurst var(--dur,1.7s) ease-out forwards;animation-delay:var(--delay,0s);}
@keyframes smokeLift{
  0%{opacity:0;transform:translate(-50%,-50%) scale(.2);}
  14%{opacity:var(--alpha,.72);}
  58%{opacity:calc(var(--alpha,.72) * .54);}
  100%{opacity:0;transform:translate(calc(-50% + var(--dx,0px)),calc(-50% + var(--dy,-160px))) scale(var(--scale,1.8));}
}
@keyframes sparkBurst{
  0%{opacity:0;transform:translate(-50%,-50%) scale(.2);}
  20%{opacity:.95;}
  100%{opacity:0;transform:translate(calc(-50% + var(--dx,0px)),calc(-50% + var(--dy,-80px))) scale(1.4);}
}
@keyframes cinematicBackdrop{
  0%{opacity:0;}
  12%{opacity:1;}
  78%{opacity:1;}
  100%{opacity:0;}
}
@keyframes cinematicCardIn{
  0%{opacity:0;transform:translateY(24px) scale(.92);}
  10%{opacity:1;transform:translateY(0) scale(1);}
  78%{opacity:1;transform:translateY(0) scale(1);}
  100%{opacity:0;transform:translateY(-12px) scale(1.02);}
}

/* MODAL */
.modal-overlay{position:fixed;top:0;left:0;right:0;bottom:0;background:rgba(0,0,0,.75);z-index:960;display:none;justify-content:center;align-items:center;}
.modal-overlay.show{display:flex;}
.modal{background:#151a30;border:1px solid #e94560;border-radius:8px;padding:18px;max-width:760px;width:94%;max-height:84vh;overflow-y:auto;box-shadow:0 8px 30px rgba(0,0,0,.6);}
.modal h2{color:#e94560;margin-bottom:10px;font-size:18px;}
.modal-close{float:right;cursor:pointer;color:#e94560;font-size:16px;padding:2px 6px;border-radius:3px;}.modal-close:hover{background:rgba(233,69,96,.2);}
.view-loader{position:fixed;top:108px;left:0;right:0;bottom:64px;z-index:860;display:none;align-items:center;justify-content:center;pointer-events:none;background:linear-gradient(180deg,rgba(1,4,10,.16),rgba(1,4,10,.24));}
.view-loader.show{display:flex;}
.view-loader-card{min-width:240px;padding:18px 22px;border-radius:18px;background:rgba(7,15,31,.9);border:1px solid rgba(255,255,255,.08);box-shadow:0 18px 40px rgba(0,0,0,.3);display:flex;align-items:center;gap:14px;color:#eef6ff;}
.view-loader-spinner{width:18px;height:18px;border-radius:50%;border:2px solid rgba(255,255,255,.12);border-top-color:#4fc3f7;animation:loaderSpin .75s linear infinite;flex-shrink:0;}
.view-loader-label{font-size:13px;font-weight:700;letter-spacing:.02em;}
body.mode-world-map .view-loader-card{background:rgba(4,11,6,.92);border-color:rgba(112,255,170,.18);}
body.mode-japan-map .view-loader-card{background:rgba(6,14,10,.92);border-color:rgba(193,230,201,.16);}
body.mode-space-map .view-loader-card{background:rgba(7,12,24,.92);border-color:rgba(114,175,255,.18);}
body.ui-loading #panelContent,body.ui-loading #gameArea canvas{filter:saturate(.92) brightness(.96);}
@keyframes loaderSpin{from{transform:rotate(0deg);}to{transform:rotate(360deg);}}

/* SLIDER */
.slider-row{display:flex;align-items:center;gap:8px;margin:6px 0;}
.slider-row label{width:120px;font-size:12px;color:#aaa;cursor:help;}
.slider-row input[type=range]{flex:1;height:6px;}
.slider-row .sv{width:70px;font-size:12px;text-align:right;color:#fff;}

/* TRADE ITEM */
.trade-item{display:flex;align-items:center;justify-content:space-between;padding:6px 0;border-bottom:1px solid rgba(255,255,255,.04);font-size:14px;gap:8px;}

/* CHART */
.mini-chart{width:100%;height:96px;background:rgba(0,0,0,.2);border-radius:6px;margin:6px 0;}

/* TABS */
.sub-tabs{display:flex;gap:2px;margin-bottom:6px;flex-wrap:wrap;}

/* SCROLL */
::-webkit-scrollbar{width:4px;}::-webkit-scrollbar-track{background:transparent;}::-webkit-scrollbar-thumb{background:#e94560;border-radius:2px;}

.bubble-warn{animation:pulse 1s infinite;}
@keyframes pulse{0%,100%{box-shadow:0 0 3px #e94560;}50%{box-shadow:0 0 12px #e94560;}}

/* INFOTIP inline */
.infotip{position:relative;display:inline-block;cursor:help;color:#f39c12;font-size:8px;margin-left:2px;border:1px solid #f39c12;border-radius:50%;width:12px;height:12px;text-align:center;line-height:11px;}
.infotip .infotip-text{display:none;position:absolute;bottom:120%;left:50%;transform:translateX(-50%);background:#1a1a3e;border:1px solid #f39c12;border-radius:4px;padding:4px 6px;font-size:8px;color:#ddd;width:180px;z-index:400;white-space:normal;line-height:1.3;}
.infotip:hover .infotip-text{display:block;}

.detail-grid{display:grid;grid-template-columns:1fr 1fr;gap:4px;}
.detail-grid .dg-item{background:rgba(0,0,0,.2);padding:4px;border-radius:3px;}
.detail-grid .dg-label{font-size:8px;color:#888;}
.detail-grid .dg-value{font-size:11px;font-weight:bold;}

.macro-grid{display:grid;grid-template-columns:1fr 1fr;gap:6px;}
.macro-card{background:rgba(0,0,0,.22);border:1px solid rgba(255,255,255,.06);border-radius:6px;padding:6px;}
.macro-card .macro-label{font-size:12px;color:#889;}
.macro-card .macro-value{font-size:19px;font-weight:bold;margin:4px 0 6px;}
.headline-card{padding:6px;margin:4px 0;background:rgba(0,0,0,.22);border-radius:5px;border-left:3px solid #4ecca3;}
.headline-card.bad{border-left-color:#e94560;}
.headline-card.good{border-left-color:#4ecca3;}
.headline-card .meta{font-size:10px;color:#888;margin-bottom:3px;}
.headline-card .body{font-size:12px;line-height:1.45;}
.save-slot{padding:8px;margin:4px 0;background:rgba(0,0,0,.22);border:1px solid rgba(255,255,255,.06);border-radius:6px;}
.save-slot .slot-title{display:flex;align-items:center;justify-content:space-between;font-size:11px;font-weight:bold;margin-bottom:4px;}
.save-slot .slot-meta{font-size:10px;color:#aaa;line-height:1.5;}
.goal-card{padding:7px;margin:5px 0;background:rgba(0,0,0,.22);border-radius:6px;border:1px solid rgba(255,255,255,.06);}
.goal-card.done{border-color:#4ecca3;box-shadow:0 0 0 1px rgba(78,204,163,.25) inset;}
.goal-title{display:flex;align-items:center;justify-content:space-between;font-size:11px;font-weight:bold;margin-bottom:4px;}
.goal-desc{font-size:11px;color:#aaa;line-height:1.5;margin-bottom:6px;}
.goal-progress{height:7px;background:rgba(255,255,255,.06);border-radius:999px;overflow:hidden;margin:5px 0;}
.goal-progress .fill{height:100%;background:linear-gradient(90deg,#4ecca3,#3498db);border-radius:999px;}
.advisor-item{padding:6px 8px;margin:4px 0;background:rgba(0,0,0,.22);border-radius:5px;font-size:10px;line-height:1.5;border-left:3px solid #3498db;}
.advisor-item.warn{border-left-color:#f39c12;}
.advisor-item.danger{border-left-color:#e94560;}
.pill{display:inline-flex;align-items:center;gap:3px;padding:2px 6px;border-radius:999px;background:rgba(255,255,255,.06);font-size:8px;color:#ccc;}
.save-actions{display:flex;gap:4px;flex-wrap:wrap;margin-top:6px;}
.focus-chip{display:inline-flex;align-items:center;gap:4px;padding:4px 10px;border-radius:999px;background:rgba(83,222,136,.12);border:1px solid rgba(97,255,164,.18);font-size:12px;color:#ddffee;}
.company-marker-label{font-size:9px;color:#ccd;}
.large-chart-wrap{background:rgba(0,0,0,.24);border:1px solid rgba(255,255,255,.08);border-radius:8px;padding:10px;margin-top:8px;}
.range-group{display:flex;gap:4px;flex-wrap:wrap;margin:6px 0;}
.compact-note{font-size:14px;color:#9aa9bf;line-height:1.7;}
.home-screen{position:fixed;inset:0;z-index:800;background:
  radial-gradient(circle at 20% 20%, rgba(52,152,219,.24), transparent 35%),
  radial-gradient(circle at 80% 18%, rgba(233,69,96,.18), transparent 32%),
  linear-gradient(160deg, rgba(5,10,20,.96), rgba(9,20,42,.97));
  display:none;align-items:flex-start;justify-content:center;padding:18px 18px 38px;overflow-y:auto;overscroll-behavior:contain;scrollbar-gutter:stable both-edges;}
.home-screen.show{display:flex;}
.home-screen{position:fixed;}
.home-screen::before{content:"";position:absolute;inset:-10%;pointer-events:none;background:
  radial-gradient(circle at 50% 18%, rgba(124,195,255,.12), transparent 24%),
  radial-gradient(circle at 18% 78%, rgba(118,255,183,.08), transparent 18%),
  radial-gradient(circle at 84% 68%, rgba(255,120,166,.08), transparent 18%);
  filter:blur(20px);opacity:.95;}
.home-screen::after{content:"";position:absolute;inset:0;pointer-events:none;background:
  linear-gradient(90deg, rgba(255,255,255,.018) 1px, transparent 1px),
  linear-gradient(rgba(255,255,255,.014) 1px, transparent 1px);
  background-size:88px 88px,88px 88px;opacity:.08;mix-blend-mode:screen;}
.home-ambient-layer{position:absolute;inset:0;pointer-events:none;overflow:hidden;z-index:0;}
.home-ambient-tile{position:absolute;border-radius:32px;border:1px solid rgba(255,255,255,.06);background-color:rgba(9,16,30,.2);background-repeat:no-repeat;background-position:center;background-size:cover;opacity:.095;filter:blur(.9px) saturate(1.15);box-shadow:0 28px 64px rgba(0,0,0,.28);mix-blend-mode:screen;}
.home-ambient-tile::after{content:"";position:absolute;inset:0;border-radius:inherit;background:linear-gradient(135deg,rgba(7,14,28,.82),rgba(7,14,28,.54) 38%,rgba(7,14,28,.9));}
.home-ambient-tile.tl{left:2%;top:8%;width:20vw;height:22vh;min-width:240px;min-height:170px;transform:rotate(-9deg);}
.home-ambient-tile.tr{right:4%;top:8%;width:22vw;height:24vh;min-width:280px;min-height:180px;transform:rotate(7deg);}
.home-ambient-tile.bl{left:4%;bottom:6%;width:24vw;height:24vh;min-width:280px;min-height:180px;transform:rotate(8deg);}
.home-ambient-tile.br{right:3%;bottom:8%;width:22vw;height:22vh;min-width:250px;min-height:170px;transform:rotate(-7deg);}
.home-ambient-tile.mid{right:12%;top:36%;width:18vw;height:20vh;min-width:220px;min-height:160px;transform:rotate(-5deg);}
.home-card{position:relative;z-index:2;width:min(1280px, 99vw);min-height:min(94vh, 1060px);display:flex;flex-direction:column;justify-content:flex-start;background:linear-gradient(180deg,rgba(9,16,30,.92),rgba(8,14,28,.9));border:1px solid rgba(255,255,255,.08);border-radius:30px;box-shadow:0 24px 70px rgba(0,0,0,.48);padding:28px 36px 38px;backdrop-filter:blur(14px);margin:10px 0 28px;}
.home-card::before{content:"";position:absolute;inset:14px;border-radius:22px;border:1px solid rgba(255,255,255,.045);pointer-events:none;}
.home-card::after{content:"";position:absolute;left:22px;right:22px;top:78px;height:120px;border-radius:26px;pointer-events:none;background:
  linear-gradient(90deg,rgba(79,195,247,.1),rgba(255,209,102,.06),rgba(176,140,255,.1)),
  repeating-linear-gradient(90deg,rgba(255,255,255,.028) 0 2px, transparent 2px 34px),
  radial-gradient(circle at 18% 50%, rgba(255,255,255,.06), transparent 32%),
  linear-gradient(115deg, rgba(255,255,255,.04), transparent 34%, rgba(255,255,255,.02) 68%, transparent 100%);
  opacity:.78;filter:blur(.2px);}
.home-title{font-size:62px;font-weight:900;letter-spacing:.035em;color:#fff;margin-bottom:10px;line-height:1.02;max-width:1000px;}
.home-sub{font-size:18px;line-height:1.84;color:#c4d4eb;max-width:1080px;}
.home-showcase{display:grid;grid-template-columns:1.74fr .58fr;gap:18px;margin-top:18px;}
.home-showcase-main,.home-showcase-side{background:rgba(255,255,255,.03);border:1px solid rgba(255,255,255,.07);border-radius:22px;padding:24px 26px;}
.home-showcase-main{background:
  radial-gradient(circle at 12% 24%, rgba(79,195,247,.1), transparent 34%),
  linear-gradient(145deg,rgba(255,255,255,.05),rgba(255,255,255,.018));
  position:relative;
  overflow:hidden;
  min-height:420px;
  padding:30px 30px 34px;
  display:flex;
  flex-direction:column;
  justify-content:space-between;
}
.home-showcase-main::before{
  content:"";
  position:absolute;
  inset:0;
  pointer-events:none;
  background:
    linear-gradient(135deg, rgba(9,16,30,.36), rgba(9,16,30,.6)),
    var(--home-hero-bg, radial-gradient(circle at 60% 40%, rgba(255,255,255,.09), transparent 44%));
  background-size:cover;
  background-position:center;
  opacity:.18;
  mix-blend-mode:screen;
  filter:blur(.8px) saturate(1.08);
}
.home-showcase-main::after{
  content:"";
  position:absolute;
  inset:18px;
  pointer-events:none;
  border-radius:18px;
  background:
    linear-gradient(90deg, rgba(255,255,255,.06) 0, transparent 18%, transparent 82%, rgba(255,255,255,.05) 100%),
    repeating-linear-gradient(90deg, rgba(255,255,255,.026) 0 2px, transparent 2px 38px),
    linear-gradient(180deg, rgba(255,255,255,.028), transparent 34%);
  opacity:.4;
}
.home-showcase-kicker{font-size:13px;letter-spacing:.18em;text-transform:uppercase;color:#87c9ff;font-weight:800;margin-bottom:10px;}
.home-showcase-value{position:relative;z-index:1;font-size:52px;font-weight:900;line-height:1.08;color:#f8fbff;max-width:860px;text-shadow:0 10px 28px rgba(0,0,0,.24);}
.home-showcase-note{position:relative;z-index:1;font-size:17px;line-height:1.82;color:#adc2dc;margin-top:14px;max-width:860px;}
.home-showcase-kicker{position:relative;z-index:1;}
.home-showcase-actions{position:relative;z-index:1;display:flex;flex-wrap:wrap;gap:14px;margin-top:28px;align-items:center;}
.home-showcase-actions .home-main-btn,.home-showcase-actions .home-sub-btn{min-width:208px;font-size:17px;min-height:62px;border-radius:24px;padding:14px 20px;}
.home-difficulty-strip{position:relative;z-index:1;margin-top:18px;padding:14px 16px;border-radius:18px;background:linear-gradient(145deg,rgba(255,255,255,.06),rgba(255,255,255,.022));border:1px solid rgba(255,255,255,.08);}
.home-difficulty-head{display:flex;align-items:center;justify-content:space-between;gap:10px;flex-wrap:wrap;margin-bottom:10px;}
.home-difficulty-head .label{font-size:12px;letter-spacing:.1em;text-transform:uppercase;color:#90a8c9;font-weight:800;}
.home-difficulty-head .value{font-size:20px;font-weight:900;color:#f4f9ff;}
.home-difficulty-actions{display:flex;flex-wrap:wrap;gap:8px;}
.home-difficulty-actions .btn{min-width:120px;}
.home-difficulty-note{margin-top:10px;font-size:13px;line-height:1.65;color:#a9bdd9;}
.home-showcase-side{display:grid;grid-template-columns:1fr;gap:10px;background:
  radial-gradient(circle at 78% 20%, rgba(255,117,171,.08), transparent 28%),
  linear-gradient(145deg,rgba(255,255,255,.04),rgba(255,255,255,.016));
}
.home-showcase-card{padding:14px 16px;border-radius:18px;border:1px solid rgba(255,255,255,.07);background:rgba(255,255,255,.028);}
.home-showcase-card .label{display:block;font-size:12px;font-weight:700;letter-spacing:.08em;text-transform:uppercase;color:#90a8c9;margin-bottom:6px;}
.home-showcase-card .value{display:block;font-size:22px;font-weight:900;color:#f6fbff;line-height:1.25;}
.home-showcase-card .note{display:block;font-size:12px;line-height:1.6;color:#9fb2cd;margin-top:6px;}
.home-grid{display:grid;grid-template-columns:1.28fr .82fr;gap:20px;margin-top:16px;align-items:stretch;}
.home-panel{background:rgba(255,255,255,.03);border:1px solid rgba(255,255,255,.07);border-radius:20px;padding:22px;}
.home-actions{display:flex;flex-wrap:wrap;gap:12px;margin-top:20px;}
.home-actions .btn{font-size:13px;padding:10px 14px;}
.home-main-btn,.home-sub-btn{
  position:relative;
  display:inline-flex;
  align-items:center;
  justify-content:center;
  gap:8px;
  min-height:52px;
  padding:12px 18px;
  border-radius:18px;
  border:1px solid rgba(255,255,255,.16);
  background:
    radial-gradient(circle at 22% 18%, rgba(255,255,255,.24), transparent 34%),
    linear-gradient(135deg,rgba(255,255,255,.08),rgba(255,255,255,.03));
  color:#eff8ff;
  font-size:14px;
  font-weight:800;
  letter-spacing:.04em;
  cursor:pointer;
  box-shadow:0 16px 30px rgba(0,0,0,.24), inset 0 1px 0 rgba(255,255,255,.12);
  transition:transform .14s ease, box-shadow .16s ease, border-color .16s ease, background .16s ease, color .16s ease;
  backdrop-filter:blur(12px);
  overflow:hidden;
}
.home-main-btn:hover,.home-sub-btn:hover{
  transform:translateY(-2px);
  box-shadow:0 20px 38px rgba(0,0,0,.28), inset 0 1px 0 rgba(255,255,255,.18);
}
.home-main-btn::after,.home-sub-btn::after{
  content:"";
  position:absolute;
  right:-26px;
  bottom:-34px;
  width:116px;
  height:116px;
  border-radius:50%;
  background:radial-gradient(circle,rgba(255,255,255,.18),rgba(255,255,255,0));
  pointer-events:none;
}
.home-main-btn{
  min-width:180px;
  background:linear-gradient(135deg,rgba(79,195,247,.22),rgba(83,222,136,.18));
  border-color:rgba(92,210,255,.42);
  color:#f3fbff;
}
.home-main-btn:hover{
  background:linear-gradient(135deg,#4fc3f7,#53de88);
  color:#03111f;
}
.home-sub-btn{
  min-width:150px;
}
#homeAutoBtn{
  background:linear-gradient(135deg,rgba(255,193,86,.18),rgba(255,138,101,.14));
  border-color:rgba(255,210,125,.34);
  color:#fff3da;
}
#homeSaveBtn{
  background:linear-gradient(135deg,rgba(176,140,255,.16),rgba(97,215,255,.14));
  border-color:rgba(190,170,255,.34);
  color:#f2eaff;
}
#homeResetBtn{
  background:linear-gradient(135deg,rgba(255,141,95,.18),rgba(255,79,123,.16));
  border-color:rgba(255,170,163,.34);
  color:#fff0f3;
}
.home-sub-btn:hover{
  color:#03111f;
}
#homeAutoBtn:hover{background:linear-gradient(135deg,#ffd166,#ff8a65);}
#homeSaveBtn:hover{background:linear-gradient(135deg,#b08cff,#61d7ff);}
#homeResetBtn:hover{background:linear-gradient(135deg,#ff8d5f,#ff4f7b);}
.home-main-btn:active,.home-sub-btn:active{transform:translateY(0);}
#homeHeroStartBtn{
  background:linear-gradient(135deg,#61d7ff,#53de88 52%,#d8ff85);
  border-color:rgba(210,255,245,.58);
  color:#06131d;
  box-shadow:0 18px 36px rgba(70,214,177,.24), inset 0 1px 0 rgba(255,255,255,.36);
}
#homeHeroStartBtn:hover{background:linear-gradient(135deg,#8ce8ff,#79f3a8 52%,#efff9b);color:#041018;}
#homeHeroContinueBtn{
  background:linear-gradient(135deg,#ffe09a,#ffb28d 54%,#ffbedf);
  border-color:rgba(255,230,176,.54);
  color:#2e1212;
  box-shadow:0 18px 36px rgba(255,174,125,.23), inset 0 1px 0 rgba(255,255,255,.3);
}
#homeHeroContinueBtn:hover{background:linear-gradient(135deg,#ffebb5,#ffc39f 54%,#ffcbe7);color:#21090a;}
#homeHeroSaveBtn{
  background:linear-gradient(135deg,#a8b8ff,#8fe7ff 56%,#9ef5ff);
  border-color:rgba(205,222,255,.46);
  color:#071322;
}
#homeHeroSaveBtn:hover{background:linear-gradient(135deg,#bec8ff,#a5efff 56%,#b5fbff);color:#04111d;}
#homeHeroResetBtn{
  background:linear-gradient(135deg,#ffbc8c,#ff8e8e 50%,#ff8dd0);
  border-color:rgba(255,203,188,.5);
  color:#2a0716;
}
#homeHeroResetBtn:hover{background:linear-gradient(135deg,#ffcaa2,#ffa5a5 50%,#ff9fe1);color:#230611;}
#homeHeroTutorialBtn{
  background:linear-gradient(135deg,#b0f3ff,#b8ffc7 48%,#fff8ad);
  border-color:rgba(180,246,255,.5);
  color:#082113;
}
#homeHeroTutorialBtn:hover{background:linear-gradient(135deg,#c4f7ff,#ccffd6 48%,#fffcc2);color:#03111f;}
.home-list{display:flex;flex-direction:column;gap:12px;margin-top:12px;font-size:14px;color:#d4dff1;line-height:1.8;}
.home-badge{display:inline-flex;align-items:center;gap:6px;padding:6px 12px;border-radius:999px;background:rgba(255,255,255,.07);font-size:12px;color:#d8e2f0;font-weight:700;}
.home-message{margin-top:18px;font-size:13px;color:#ffcc80;min-height:20px;}
.home-setting-row{display:flex;align-items:center;justify-content:space-between;gap:10px;padding:12px 0;border-bottom:1px solid rgba(255,255,255,.06);font-size:14px;}
.home-setting-row:last-child{border-bottom:none;}
.home-setting-col{display:flex;flex-direction:column;align-items:flex-end;gap:8px;}
.home-lang{display:flex;gap:8px;flex-wrap:wrap;margin-top:16px;}
.home-save-list{display:flex;flex-direction:column;gap:10px;margin-top:16px;}
.home-save-item{display:flex;justify-content:space-between;align-items:center;gap:8px;padding:12px 14px;border:1px solid rgba(255,255,255,.07);border-radius:14px;background:rgba(255,255,255,.03);}
.home-save-list.collapsed{display:none;}
.save-slot-name{font-size:14px;font-weight:800;color:#f7fbff;}
.title-fun-layer{position:absolute;inset:0;pointer-events:none;overflow:hidden;}
.title-sign{position:absolute;pointer-events:auto;padding:10px 16px;border-radius:18px;background:linear-gradient(135deg,rgba(255,255,255,.18),rgba(255,255,255,.06));border:1px solid rgba(255,255,255,.22);box-shadow:0 16px 34px rgba(0,0,0,.28), inset 0 1px 0 rgba(255,255,255,.28);color:#fff;font-size:14px;font-weight:800;letter-spacing:.04em;cursor:pointer;user-select:none;transition:transform .14s ease, opacity .14s ease, box-shadow .14s ease;backdrop-filter:blur(12px);text-shadow:0 1px 0 rgba(0,0,0,.22);}
.title-sign:hover{transform:translateY(-2px) rotate(-2deg) scale(1.02);box-shadow:0 20px 40px rgba(0,0,0,.34), inset 0 1px 0 rgba(255,255,255,.34);}
.title-sign[data-tone="economy"]{background:linear-gradient(135deg,#ff8d5f,#ff4f7b);border-color:rgba(255,198,172,.48);}
.title-sign[data-tone="market"]{background:linear-gradient(135deg,#61d7ff,#3977ff);border-color:rgba(189,237,255,.46);}
.title-sign[data-tone="public"]{background:linear-gradient(135deg,#7effb4,#15b97e);border-color:rgba(194,255,218,.4);}
.title-sign[data-tone="space"]{background:linear-gradient(135deg,#b08cff,#ff7ae3);border-color:rgba(233,205,255,.44);}
.title-sign.falling{animation:titleSignFall 1.35s cubic-bezier(.2,.78,.18,1) forwards;}
@keyframes titleSignFall{
  0%{transform:translateY(0) rotate(0deg);opacity:1;}
  70%{opacity:1;}
  100%{transform:translateY(120vh) rotate(28deg);opacity:0;}
}
.home-title-tools{display:flex;gap:10px;flex-wrap:wrap;margin-top:12px;}
.home-title-tools .btn{font-size:13px;padding:8px 12px;}
.history-legend{display:flex;justify-content:space-between;gap:12px;flex-wrap:wrap;font-size:11px;color:#a6b5cc;margin-top:8px;}
.history-hover-readout{display:flex;justify-content:space-between;gap:10px;flex-wrap:wrap;font-size:12px;color:#d8e5f7;margin-bottom:8px;padding:8px 10px;border-radius:10px;background:rgba(255,255,255,.03);border:1px solid rgba(255,255,255,.06);}
.history-chart-wrap{position:relative;width:100%;height:240px;border-radius:8px;overflow:hidden;background:rgba(0,0,0,.22);}
.history-chart-svg{position:absolute;inset:0;width:100%;height:100%;}
.history-cursor-line{position:absolute;top:10px;bottom:10px;width:1px;background:rgba(255,255,255,.22);pointer-events:none;opacity:0;transform:translateX(-50%);}
.history-point-marker{position:absolute;width:10px;height:10px;border-radius:50%;border:2px solid #fff;background:#0f1320;transform:translate(-50%,-50%);pointer-events:none;opacity:0;box-shadow:0 0 0 3px rgba(255,255,255,.08);}
.top-stat-chart-wrap{height:190px;}
.top-stat-chart-wrap .history-cursor-line{top:12px;bottom:12px;}
.top-stat-chart-wrap .history-point-marker{width:12px;height:12px;}
.history-axis-note{display:flex;justify-content:space-between;gap:8px;flex-wrap:wrap;font-size:11px;color:#b8c6da;margin-top:8px;}
.history-axis-note strong{color:#eff8ff;font-weight:700;}
.research-card{padding:10px;margin:8px 0;background:rgba(0,0,0,.22);border-radius:10px;border:1px solid rgba(255,255,255,.07);}
.research-card.locked{opacity:.72;}
.research-hero{position:relative;height:146px;margin:-10px -10px 10px;border-radius:10px 10px 0 0;overflow:hidden;background:linear-gradient(135deg,rgba(64,128,255,.25),rgba(12,20,44,.88));border-bottom:1px solid rgba(255,255,255,.08);}
.research-hero img{width:100%;height:100%;display:block;object-fit:cover;filter:saturate(1.04) contrast(1.02);}
.research-hero::after{content:"";position:absolute;inset:0;background:linear-gradient(180deg,rgba(3,6,12,.04),rgba(3,6,12,.12) 40%,rgba(3,6,12,.74) 100%);}
.research-hero-badge{position:absolute;left:12px;bottom:10px;z-index:2;padding:5px 9px;border-radius:999px;background:rgba(6,13,24,.76);border:1px solid rgba(255,255,255,.12);color:#eef6ff;font-size:11px;font-weight:800;letter-spacing:.04em;backdrop-filter:blur(8px);}
.research-hero-badge.pathogen{background:rgba(48,8,14,.78);border-color:rgba(233,69,96,.25);}
.research-card .title{display:flex;justify-content:space-between;gap:8px;font-size:12px;font-weight:700;margin-bottom:6px;}
.research-card .meta{font-size:12px;color:#96a2b5;line-height:1.6;}
.research-card .status{font-size:12px;color:#ffd180;margin-top:6px;}
.research-card .actions{display:flex;flex-wrap:wrap;gap:6px;margin-top:10px;}
.market-filter-btn{position:relative;}
.market-filter-btn[title]{cursor:help;}
.drag-project-pill{display:inline-flex;align-items:center;gap:6px;padding:4px 8px;border-radius:999px;background:rgba(243,156,18,.15);color:#ffd180;font-size:10px;margin-top:6px;}
.map-hud{position:fixed;top:116px;left:368px;z-index:125;display:flex;flex-direction:column;gap:8px;pointer-events:none;transition:transform .16s ease;}
.map-hud-body{display:flex;flex-direction:column;gap:8px;transition:opacity .18s ease, transform .18s ease, max-height .18s ease;}
.map-hud-row{display:flex;gap:6px;flex-wrap:wrap;align-items:center;padding:8px 10px;border:1px solid rgba(255,255,255,.08);border-radius:16px;background:rgba(8,16,18,.72);backdrop-filter:blur(12px);box-shadow:0 10px 25px rgba(0,0,0,.18);pointer-events:auto;transition:opacity .16s ease, max-height .18s ease, padding .18s ease, border-width .18s ease, box-shadow .18s ease;max-height:120px;overflow:hidden;}
.map-hud-row .btn{padding:6px 10px;}
.map-hud-row.secondary{gap:8px;max-width:min(640px,60vw);}
.map-hud-row.tertiary{display:none;max-width:min(720px,64vw);align-items:flex-start;}
.map-hud.trace-open .map-hud-row.tertiary{display:flex;}
.map-hud-row .pill{font-size:11px;background:rgba(255,255,255,.08);}
.map-trace-panel{display:flex;flex-direction:column;gap:8px;width:min(680px,58vw);}
.map-trace-toolbar{display:flex;flex-wrap:wrap;gap:6px;align-items:center;}
.map-trace-toolbar .pill{font-size:11px;}
.map-trace-output{width:100%;min-height:110px;border-radius:12px;border:1px solid rgba(255,255,255,.08);background:rgba(5,12,14,.86);color:#eff8f1;padding:10px 12px;font-size:12px;line-height:1.5;resize:vertical;}
.map-trace-hint{font-size:12px;color:#b8c9c1;line-height:1.5;}
.prefecture-invest-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:10px;margin-top:10px;}
.prefecture-invest-card{padding:12px;border-radius:14px;background:rgba(255,255,255,.04);border:1px solid rgba(255,255,255,.06);}
.prefecture-invest-card .title{font-size:15px;font-weight:800;color:#eff7f0;}
.prefecture-invest-card .meta{font-size:12px;color:#a9bcb0;line-height:1.6;margin-top:6px;}
.prefecture-invest-card .actions{display:flex;flex-wrap:wrap;gap:6px;margin-top:10px;}
.map-hud-fold-btn{display:inline-flex;align-items:center;justify-content:center;position:absolute;top:12px;right:-18px;width:40px;height:40px;border-radius:999px;border:1px solid rgba(255,255,255,.14);background:rgba(7,15,29,.72);color:#f4fbff;font-size:15px;font-weight:800;backdrop-filter:blur(10px);cursor:pointer;pointer-events:auto;box-shadow:0 10px 24px rgba(0,0,0,.24);transition:transform .16s ease, background .16s ease, right .16s ease, top .16s ease;}
.map-hud-fold-btn:hover{background:rgba(12,24,42,.88);}
.map-hud-drag-btn{display:inline-flex;align-items:center;justify-content:center;position:absolute;top:12px;left:-18px;width:40px;height:40px;border-radius:999px;border:1px solid rgba(255,255,255,.14);background:rgba(7,15,29,.72);color:#f4fbff;font-size:15px;font-weight:800;backdrop-filter:blur(10px);cursor:grab;pointer-events:auto;box-shadow:0 10px 24px rgba(0,0,0,.24);transition:transform .16s ease, background .16s ease, left .16s ease, top .16s ease;}
.map-hud-drag-btn:hover{background:rgba(12,24,42,.88);}
.map-hud-drag-btn:active{cursor:grabbing;}
.map-hud.collapsed{pointer-events:auto;gap:0;}
.map-hud.collapsed .map-hud-body{opacity:0;transform:translateX(-12px);max-height:0;overflow:hidden;pointer-events:none;}
.map-hud.collapsed .map-hud-row{opacity:0;max-height:0;padding-top:0;padding-bottom:0;border-width:0;box-shadow:none;}
.map-hud.collapsed .map-hud-fold-btn{top:10px;right:-16px;}
.map-hud.collapsed .map-hud-drag-btn{top:10px;left:-16px;}
.battle-screen{display:flex;flex-direction:column;gap:12px;}
.battle-hero{padding:14px 16px;border-radius:18px;background:linear-gradient(135deg,rgba(12,22,44,.94),rgba(9,17,34,.92));border:1px solid rgba(255,255,255,.09);box-shadow:0 16px 32px rgba(0,0,0,.28);}
.battle-hero .title{font-size:18px;font-weight:800;color:#eff7ff;}
.battle-hero .subtitle{font-size:13px;color:#9eb1c8;line-height:1.6;margin-top:6px;}
.battle-vs-grid{display:grid;grid-template-columns:1fr auto 1fr;gap:12px;align-items:stretch;}
.battle-side{padding:12px;border-radius:16px;background:rgba(0,0,0,.2);border:1px solid rgba(255,255,255,.08);}
.battle-side.jp{box-shadow:inset 0 0 0 1px rgba(97,183,255,.2);}
.battle-side.enemy{box-shadow:inset 0 0 0 1px rgba(255,122,122,.18);}
.battle-side .label{font-size:13px;color:#9fb3ca;margin-bottom:6px;}
.battle-side .power{font-size:28px;font-weight:900;margin-bottom:8px;}
.battle-side.jp .power{color:#7ec9ff;}
.battle-side.enemy .power{color:#ff9b90;}
.battle-side .meta{font-size:13px;line-height:1.6;color:#dfe8f5;}
.battle-versus{display:flex;align-items:center;justify-content:center;font-size:24px;font-weight:900;color:#ffd180;padding:0 6px;}
.battle-actions{display:flex;flex-wrap:wrap;gap:8px;}
.battle-reinforcement-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:10px;}
.battle-reinforcement-card{padding:10px;border-radius:14px;background:rgba(255,255,255,.03);border:1px solid rgba(255,255,255,.07);}
.battle-reinforcement-card .top{display:flex;justify-content:space-between;gap:8px;font-size:14px;font-weight:700;margin-bottom:8px;}
.battle-log{max-height:220px;overflow-y:auto;padding-right:4px;}
.military-art-card{display:grid;grid-template-columns:minmax(150px,210px) 1fr;gap:12px;align-items:stretch;}
.military-art-visual{position:relative;min-height:140px;border-radius:14px;overflow:hidden;border:1px solid rgba(255,255,255,.1);background:linear-gradient(180deg,rgba(255,255,255,.04),rgba(0,0,0,.18));box-shadow:inset 0 1px 0 rgba(255,255,255,.04);}
.military-art-visual::after{content:"";position:absolute;inset:0;background:linear-gradient(180deg,rgba(0,0,0,.02),rgba(0,0,0,.34));pointer-events:none;}
.military-art-visual .art{position:absolute;inset:0;background-size:cover;background-position:center;transform:scale(1.02);}
.military-art-visual .badge{position:absolute;left:10px;bottom:10px;z-index:2;padding:6px 9px;border-radius:999px;background:rgba(6,12,22,.78);border:1px solid rgba(255,255,255,.16);font-size:11px;font-weight:800;color:#eff7ff;backdrop-filter:blur(8px);}
.military-art-details{display:flex;flex-direction:column;gap:8px;}
.military-art-sub{font-size:11px;color:#9fb3ca;line-height:1.6;}
.trade-slider-wrap{padding:10px 12px;border-radius:14px;background:rgba(255,255,255,.03);border:1px solid rgba(255,255,255,.07);margin:8px 0 12px;}
.trade-slider-head{display:flex;align-items:center;justify-content:space-between;gap:12px;font-size:14px;font-weight:700;margin-bottom:8px;}
.trade-slider-value{font-size:20px;font-weight:900;color:#ffd180;}
.trade-slider-wrap input[type=range]{width:100%;height:7px;accent-color:#f39c12;}
body.focus-ui #topBar,
body.focus-ui #priceBar,
body.focus-ui #eventLog,
body.focus-ui #bottomBar{display:none!important;}
body.focus-ui .floating-dock{display:none!important;}
body.focus-ui #gameArea{top:0;bottom:0;}
body.focus-ui #rightPanel{top:0;bottom:0;}
body.focus-ui .map-hud{top:10px!important;}
body.focus-ui .view-loader{top:0;bottom:0;}
.modal.modal-world-country{max-width:min(1120px,96vw);width:96vw;}
.modal.modal-cyber{max-width:min(1320px,98vw);width:98vw;max-height:94vh;height:94vh;padding:10px 12px;}
.modal.modal-company{max-width:min(1280px,98vw);width:98vw;max-height:94vh;}
.modal.modal-company .detail-grid{grid-template-columns:repeat(3,minmax(0,1fr));gap:10px;}
.modal.modal-company .dg-item{padding:10px;border-radius:12px;}
.modal.modal-company .dg-label{font-size:11px;}
.modal.modal-company .dg-value{font-size:18px;line-height:1.45;font-weight:800;}
.company-peak-banner{margin:8px 0;padding:10px 14px;border-radius:14px;background:linear-gradient(90deg,rgba(255,213,79,.18),rgba(255,94,98,.18));border:1px solid rgba(255,214,102,.36);font-size:13px;font-weight:800;color:#fff0c8;box-shadow:0 10px 24px rgba(0,0,0,.18);}
.modal.modal-prefecture{max-width:min(1040px,94vw);width:94vw;max-height:90vh;}
.prefecture-hero{padding:14px 16px;border-radius:18px;background:linear-gradient(135deg,rgba(16,36,24,.94),rgba(8,18,12,.92));border:1px solid rgba(193,230,201,.16);box-shadow:0 16px 30px rgba(0,0,0,.22);margin-bottom:10px;}
.prefecture-hero .title{font-size:30px;font-weight:900;color:#f4fff7;letter-spacing:.04em;}
.prefecture-hero .sub{font-size:14px;color:#cae2d1;line-height:1.7;margin-top:6px;}
.focus-tray{position:fixed;z-index:155;display:none;pointer-events:none;}
body.focus-ui .focus-tray{display:flex;}
.focus-tray.focus-tray-left{left:14px;top:50%;transform:translateY(-50%);flex-direction:row;align-items:center;gap:10px;}
.focus-tray.focus-tray-bottom{left:50%;bottom:14px;transform:translateX(-50%);flex-direction:column;align-items:center;gap:10px;}
.focus-tray-toggle{pointer-events:auto;width:44px;height:44px;border-radius:999px;border:1px solid rgba(255,255,255,.18);background:rgba(7,15,29,.55);color:#eff8ff;backdrop-filter:blur(12px);font-size:18px;font-weight:900;cursor:pointer;box-shadow:0 12px 26px rgba(0,0,0,.28);transition:transform .14s ease, background .14s ease;}
.focus-tray-toggle:hover{transform:scale(1.05);background:rgba(10,24,43,.72);}
.focus-tray-panel{pointer-events:auto;display:flex;gap:10px;transition:transform .18s ease, opacity .18s ease;opacity:0;}
.focus-tray.focus-tray-left .focus-tray-panel{flex-direction:column;transform:translateX(-18px);}
.focus-tray.focus-tray-bottom .focus-tray-panel{flex-direction:row;flex-wrap:wrap;justify-content:center;transform:translateY(18px);max-width:min(92vw,1080px);}
.focus-tray.open .focus-tray-panel{opacity:1;transform:none;}
.focus-tray:not(.open) .focus-tray-panel{pointer-events:none;}
.focus-tray.focus-tray-left .focus-tray-panel{max-height:calc(100vh - 120px);overflow:auto;padding-right:6px;}
.focus-tray-card{min-width:168px;max-width:220px;padding:10px 12px;border-radius:16px;background:rgba(7,15,29,.72);border:1px solid rgba(255,255,255,.1);backdrop-filter:blur(14px);box-shadow:0 16px 30px rgba(0,0,0,.26);cursor:pointer;transition:transform .14s ease, border-color .14s ease, background .14s ease;}
.focus-tray-card:hover{transform:translateY(-2px);border-color:rgba(111,195,255,.28);background:rgba(9,19,36,.84);}
.focus-tray-card .lbl{font-size:11px;color:#9eb6d5;letter-spacing:.04em;text-transform:uppercase;}
.focus-tray-card .val{font-size:21px;font-weight:900;color:#f3fbff;margin-top:4px;}
.focus-tray-card .rate{font-size:11px;margin-top:4px;}
.focus-tray-card .hint{font-size:10px;color:#86a1bf;margin-top:5px;}
.focus-tray-empty{padding:12px 14px;border-radius:16px;background:rgba(7,15,29,.72);border:1px solid rgba(255,255,255,.1);font-size:12px;color:#cfe2f5;max-width:260px;}
.focus-time-dock{position:fixed;left:14px;bottom:14px;z-index:154;display:none;align-items:center;gap:8px;padding:10px 12px;border-radius:18px;background:rgba(7,15,29,.72);border:1px solid rgba(255,255,255,.1);backdrop-filter:blur(16px);box-shadow:0 18px 36px rgba(0,0,0,.28);}
body.focus-ui .focus-time-dock{display:flex;}
.focus-time-dock .dock-group{display:flex;align-items:center;gap:6px;flex-wrap:wrap;}
.focus-time-dock .dock-divider{width:1px;height:30px;background:rgba(255,255,255,.12);border-radius:999px;}
.focus-time-dock .dock-label{font-size:11px;color:#9eb6d5;font-weight:700;letter-spacing:.05em;text-transform:uppercase;margin-right:2px;}
.focus-time-dock .dock-btn{min-width:44px;padding:8px 10px;border-radius:12px;border:1px solid rgba(255,255,255,.12);background:rgba(255,255,255,.03);color:#edf6ff;cursor:pointer;font-size:13px;font-weight:700;transition:transform .12s ease, background .12s ease, border-color .12s ease, color .12s ease;}
.focus-time-dock .dock-btn:hover{transform:translateY(-1px);background:rgba(255,255,255,.08);border-color:rgba(255,255,255,.2);}
.focus-time-dock .dock-btn.active{background:rgba(233,69,96,.18);border-color:rgba(233,69,96,.5);color:#fff;}
.focus-time-dock .dock-btn.btn-turn-active{background:rgba(243,156,18,.18);border-color:rgba(243,156,18,.48);color:#fff3d0;}
.focus-time-dock .dock-btn:disabled{opacity:.45;cursor:not-allowed;transform:none;}
.mode-world-map .focus-time-dock{background:rgba(4,11,6,.78);border-color:rgba(111,255,171,.18);}
.mode-japan-map .focus-time-dock{background:rgba(7,16,12,.8);border-color:rgba(193,230,201,.18);}
.mode-space-map .focus-time-dock{background:rgba(8,13,28,.82);border-color:rgba(114,175,255,.2);}
.world-country-sheet{display:flex;flex-direction:column;gap:12px;}
.world-country-sheet .detail-grid{grid-template-columns:repeat(3,minmax(0,1fr));gap:8px;}
.world-country-sheet .dg-item{padding:8px;border-radius:10px;}
.world-country-sheet .dg-label{font-size:10px;}
.world-country-sheet .dg-value{font-size:15px;line-height:1.4;}
.world-country-hero{padding:14px 16px;border-radius:18px;background:linear-gradient(135deg,rgba(18,36,69,.94),rgba(10,18,32,.94));border:1px solid rgba(120,199,255,.18);box-shadow:0 16px 30px rgba(0,0,0,.22);}
.world-country-hero .title{font-size:24px;font-weight:900;color:#f3fbff;}
.world-country-hero .sub{font-size:13px;color:#b7d0ea;line-height:1.7;margin-top:6px;}
.world-country-actions{display:flex;gap:8px;flex-wrap:wrap;}
.mode-world-map .map-hud-row{background:rgba(4,11,6,.86);border:1px solid rgba(111,255,171,.2);box-shadow:0 10px 28px rgba(0,0,0,.32);}
.mode-world-map .map-hud-row.secondary{background:rgba(3,8,5,.82);}
.mode-world-map .map-hud-row .pill{background:rgba(91,255,158,.12);color:#e8fff2;}
.mode-japan-map .map-hud-row{background:rgba(7,16,12,.84);border:1px solid rgba(193,230,201,.18);box-shadow:0 10px 28px rgba(0,0,0,.28);}
.mode-japan-map .map-hud-row.secondary{background:rgba(6,12,10,.82);}
.mode-japan-map .map-hud-row .pill{background:rgba(193,230,201,.12);color:#f4fff7;}
.mode-space-map .map-hud-row{background:rgba(8,13,28,.86);border:1px solid rgba(114,175,255,.2);box-shadow:0 10px 28px rgba(0,0,0,.32);}
.mode-space-map .map-hud-row.secondary{background:rgba(6,10,20,.82);}
.mode-space-map .map-hud-row .pill{background:rgba(114,175,255,.12);color:#eef6ff;}
.world-country-card{padding:12px;margin:8px 0;background:linear-gradient(180deg,rgba(95,255,158,.05),rgba(255,255,255,.015));border-radius:14px;border:1px solid rgba(111,255,171,.12);box-shadow:0 12px 26px rgba(0,0,0,.18);}
.world-country-card .top{display:flex;justify-content:space-between;align-items:center;gap:8px;margin-bottom:6px;}
.world-country-card .name{font-size:13px;font-weight:700;color:#effff4;}
.world-country-card .meta{font-size:10px;color:#9fcfb0;line-height:1.6;}
.world-country-card .bars{display:grid;grid-template-columns:repeat(3,1fr);gap:6px;margin-top:8px;}
.world-country-card .barbox{background:rgba(0,0,0,.26);border:1px solid rgba(111,255,171,.08);border-radius:10px;padding:7px;}
.world-country-card .barlabel{font-size:8px;color:#8bc79b;text-transform:uppercase;letter-spacing:.06em;}
.world-country-card .barvalue{font-size:13px;font-weight:700;margin-top:3px;}
.map-note-card{padding:12px 14px;margin-bottom:10px;background:linear-gradient(135deg,rgba(69,197,110,.16),rgba(105,255,171,.07));border:1px solid rgba(111,255,171,.18);border-radius:16px;box-shadow:inset 0 1px 0 rgba(255,255,255,.03);}
.map-note-card .title{font-size:13px;font-weight:700;color:#effff4;margin-bottom:4px;}
.map-note-card .body{font-size:11px;color:#b2d7bf;line-height:1.7;}
.world-sector-list{display:flex;gap:6px;flex-wrap:wrap;margin-top:8px;}
.world-sector-pill{display:inline-flex;align-items:center;gap:4px;padding:5px 9px;border-radius:999px;background:rgba(96,255,156,.08);border:1px solid rgba(96,255,156,.14);font-size:10px;color:#effff4;}
.world-link-row{display:flex;gap:6px;flex-wrap:wrap;margin-top:8px;}
.status-badge{display:inline-flex;align-items:center;gap:4px;padding:4px 8px;border-radius:999px;background:rgba(255,255,255,.07);font-size:11px;color:#d9e6f6;}
.status-badge.good{background:rgba(78,204,163,.12);color:#93f3cf;}
.status-badge.warn{background:rgba(255,193,86,.12);color:#ffd27d;}
.status-badge.bad{background:rgba(233,92,96,.12);color:#ffb4b7;}
.work-card{padding:10px;margin:8px 0;background:rgba(0,0,0,.22);border-radius:10px;border:1px solid rgba(255,255,255,.06);}
.work-row{display:flex;justify-content:space-between;gap:10px;align-items:flex-start;flex-wrap:wrap;}
.work-meta{font-size:12px;color:#a7b6cd;line-height:1.6;}
.support-row{padding:10px;margin-top:8px;background:rgba(255,255,255,.03);border:1px solid rgba(255,255,255,.06);border-radius:10px;}
.foreign-stock-card{padding:10px;margin:8px 0;background:rgba(0,0,0,.22);border-radius:10px;border-left:4px solid #90caf9;}
.roadmap-stage{padding:8px 10px;margin:6px 0;border-radius:8px;background:rgba(255,255,255,.03);border:1px solid rgba(255,255,255,.06);font-size:12px;line-height:1.6;}
.roadmap-stage.done{border-color:rgba(78,204,163,.35);background:rgba(78,204,163,.08);}
.roadmap-stage.active{border-color:rgba(255,193,86,.35);background:rgba(255,193,86,.08);}
.resource-jump-btn{padding:4px 8px;font-size:11px;border-radius:999px;border:1px solid rgba(255,255,255,.12);background:rgba(255,255,255,.03);color:#dce9fa;cursor:pointer;}
.resource-jump-btn:hover{background:#3498db;color:#fff;}
.hold-hint{font-size:11px;color:#8ea0bc;margin-top:4px;}
.modal.modal-finance{max-width:min(1180px,96vw);width:min(1180px,96vw);}
.finance-grid{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:14px;margin-top:12px;}
.finance-card{padding:14px;border-radius:16px;background:linear-gradient(180deg,rgba(255,255,255,.04),rgba(255,255,255,.02));border:1px solid rgba(255,255,255,.08);box-shadow:0 14px 28px rgba(0,0,0,.18);}
.finance-card h3{font-size:16px;margin-bottom:8px;}
.finance-total{font-size:20px;font-weight:800;color:#fff;margin-bottom:10px;}
.finance-donut{width:100%;max-width:320px;margin:0 auto 10px;}
.finance-legend{display:grid;gap:8px;margin-top:8px;}
.finance-legend-row{display:grid;grid-template-columns:auto 1fr auto auto;gap:8px;align-items:center;font-size:13px;color:#dfeaff;}
.finance-legend-swatch{width:10px;height:10px;border-radius:999px;}
.finance-summary-grid{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:10px;margin-top:12px;}
.finance-summary-box{padding:10px 12px;border-radius:12px;background:rgba(0,0,0,.22);border:1px solid rgba(255,255,255,.06);}
.finance-summary-box .label{font-size:11px;color:#9cb0c8;text-transform:uppercase;letter-spacing:.06em;}
.finance-summary-box .value{font-size:18px;font-weight:800;margin-top:4px;}
.mobile-ui-toggle{display:none;position:fixed;z-index:170;align-items:center;justify-content:center;width:38px;height:38px;border-radius:999px;border:1px solid rgba(255,255,255,.16);background:rgba(6,10,18,.62);color:#f3fbff;backdrop-filter:blur(10px);box-shadow:0 10px 24px rgba(0,0,0,.25);cursor:pointer;}
.mobile-ui-toggle:hover{background:rgba(233,69,96,.88);}
#mobileTopToggle{top:12px;right:12px;}
#mobileBottomToggle{bottom:12px;left:12px;}
#mobilePanelToggle{top:50%;right:12px;transform:translateY(-50%);}
body.mobile-ui .mobile-ui-toggle{display:flex;}
body.mobile-ui #topBar,
body.mobile-ui #priceBar,
body.mobile-ui #bottomBar{transition:transform .16s ease, opacity .16s ease;}
body.mobile-ui #topBar{height:54px;padding:0 8px;gap:2px;}
body.mobile-ui #topBar .stat{min-width:76px;padding:3px 6px;}
body.mobile-ui #topBar .stat .lbl{font-size:9px;}
body.mobile-ui #topBar .stat .val{font-size:14px;}
body.mobile-ui #topBar .stat .rate{font-size:9px;}
body.mobile-ui #weekDisplay{font-size:15px;padding:0 8px;}
body.mobile-ui #priceBar{top:54px;height:34px;padding:0 8px;gap:8px;}
body.mobile-ui #gameArea{top:88px;right:0;bottom:54px;}
body.mobile-ui #rightPanel{top:88px;width:min(86vw,420px);bottom:54px;transition:transform .16s ease, width .12s ease, top .12s ease, bottom .12s ease;}
body.mobile-ui #panelTabs{gap:6px;padding:8px;}
body.mobile-ui #panelTabs .btn{font-size:12px;padding:7px 10px;}
body.mobile-ui #panelContent{padding:10px 10px 18px;}
body.mobile-ui #bottomBar{height:54px;padding:0 8px;gap:6px;}
body.mobile-ui #bottomBar .btn{padding:6px 10px;font-size:13px;}
body.mobile-ui #speedCtrl{gap:2px;}
body.mobile-ui #speedCtrl .btn{padding:6px 8px;font-size:12px;}
body.mobile-ui #eventLog{top:96px;left:8px;width:min(300px,48vw);max-height:34vh;}
body.mobile-ui .map-hud{left:8px;right:8px;top:96px;max-width:calc(100vw - 16px);}
body.mobile-ui .map-hud-row{gap:6px;padding:8px 10px;}
body.mobile-ui .map-hud-row .pill{font-size:11px;}
body.mobile-ui .floating-dock{width:min(82vw,340px);}
body.mobile-ui.mobile-top-collapsed #topBar,
body.mobile-ui.mobile-top-collapsed #priceBar{transform:translateY(-120%);opacity:0;pointer-events:none;}
body.mobile-ui.mobile-top-collapsed #gameArea,
body.mobile-ui.mobile-top-collapsed #rightPanel{top:8px;}
body.mobile-ui.mobile-top-collapsed #eventLog{top:16px;}
body.mobile-ui.mobile-top-collapsed .map-hud{top:16px;}
body.mobile-ui.mobile-bottom-collapsed #bottomBar{transform:translateY(calc(100% - 16px));}
body.mobile-ui.mobile-bottom-collapsed #gameArea,
body.mobile-ui.mobile-bottom-collapsed #rightPanel{bottom:16px;}
body.mobile-ui.mobile-panel-collapsed #rightPanel{transform:translateX(calc(100% - 16px));}
body.mobile-ui.mobile-panel-collapsed .map-hud{max-width:calc(100vw - 32px);}
body.quality-low *{scroll-behavior:auto!important;}
body.quality-low .event-msg,
body.quality-low .title-sign,
body.quality-low .floating-dock,
body.quality-low .map-hud-row,
body.quality-low #rightPanel,
body.quality-low #topBar,
body.quality-low #bottomBar{transition:none!important;animation:none!important;}
body.quality-low .view-loader-card,
body.quality-low .floating-dock,
body.quality-low .map-hud-row{backdrop-filter:none!important;}
body.quality-low #gameArea::before{opacity:.008;}
body.quality-low .home-ambient-layer,
body.quality-low .cinematic-smoke,
body.quality-low #cinematicEventBackdrop{display:none!important;}
body.quality-low .headline-card,
body.quality-low .goal-card,
body.quality-low .research-card,
body.quality-low .macro-card,
body.quality-low .company{box-shadow:none!important;}
body.quality-medium #gameArea::before{opacity:.028;}
@media (max-width: 860px){
  .home-grid{grid-template-columns:1fr;}
  .home-showcase{grid-template-columns:1fr;}
  .home-card{padding:24px 22px;width:min(96vw, 960px);min-height:auto;}
  .home-title{font-size:40px;}
  .home-showcase-value{font-size:27px;}
  .map-hud{left:10px;right:10px;}
  .map-hud-row.secondary{max-width:none;}
  .finance-grid{grid-template-columns:1fr;}
  .finance-summary-grid{grid-template-columns:repeat(2,minmax(0,1fr));}
}
</style>
</head>
<body>

<!-- TOP BAR -->
<div id="topBar">
  <div class="stat clickable" data-stat="money" onclick="handleTopStatCardClick('money')"><span class="lbl">💰 国庫</span><span class="val" id="money">90円</span><span class="rate" id="moneyRate"></span></div>
  <div class="stat clickable" data-stat="polCap" onclick="handleTopStatCardClick('polCap')"><span class="lbl">🏛 政治資本</span><span class="val" id="polCap">100/400</span><span class="rate" id="polCapRate"></span></div>
  <div class="stat clickable" data-stat="pop" onclick="handleTopStatCardClick('pop')"><span class="lbl">👥 人口</span><span class="val" id="pop">12,600万</span><span class="rate" id="popRate"></span></div>
  <div class="stat clickable" data-stat="gdp" onclick="handleTopStatCardClick('gdp')"><span class="lbl">📈 GDP</span><span class="val" id="gdp">5,400兆</span><span class="rate" id="gdpRate"></span></div>
  <div class="stat clickable" data-stat="approval" onclick="handleTopStatCardClick('approval')"><span class="lbl">😊 支持率</span><span class="val" id="approval">55%</span><span class="rate" id="approvalRate"></span></div>
  <div class="stat clickable" data-stat="bubble" onclick="handleTopStatCardClick('bubble')"><span class="lbl">🎈 バブル</span><span class="val" id="bubbleIdx">0%</span></div>
  <div class="stat clickable" data-stat="yen" onclick="handleTopStatCardClick('yen')"><span class="lbl">💱 円指数</span><span class="val" id="yenValue">100</span><span class="rate" id="yenRate"></span></div>
  <div class="stat clickable" data-stat="tech" onclick="handleTopStatCardClick('tech')"><span class="lbl">🧠 技術力</span><span class="val" id="techLvl">12</span><span class="rate" id="techRate"></span></div>
  <div class="stat clickable" data-stat="defense" onclick="handleTopStatCardClick('defense')"><span class="lbl">🛡 防衛力</span><span class="val" id="defPow">10</span></div>
  <div class="stat clickable" data-stat="cyber" onclick="handleTopStatCardClick('cyber')"><span class="lbl">💻 サイバー</span><span class="val" id="cyberPow">5</span></div>
  <div class="stat clickable" data-stat="nuclear" onclick="handleTopStatCardClick('nuclear')"><span class="lbl">☢ 核</span><span class="val" id="nukePow">0</span></div>
  <div class="stat"><span class="lbl">🧪 試験</span><span class="val" id="sandboxState">OFF</span></div>
  <div id="weekDisplay">2025年 第1週</div>
</div>

<!-- PRICE BAR -->
<div id="priceBar"></div>

<!-- EVENT LOG -->
<div class="floating-dock" id="newsDock">
  <div class="floating-dock-header" id="newsDockHeader">
    <span class="floating-dock-title" id="newsDockTitle">📰 ニュース</span>
    <div class="floating-dock-actions">
      <button class="floating-dock-handle" id="newsDockMove">⠿</button>
      <button class="floating-dock-collapse" id="newsDockCollapse" onclick="toggleFloatingDock('news')">▾</button>
    </div>
  </div>
<div class="floating-dock-body" id="newsDockBody">
  <div id="eventLog"></div>
</div>
</div>
<div id="majorEventBanner"></div>
<div id="cinematicEvent" aria-hidden="true">
  <div id="cinematicEventBackdrop"></div>
  <div id="cinematicEventCard">
    <div class="cinematic-badge" id="cinematicEventBadge">EVENT</div>
    <div class="cinematic-smoke" id="cinematicSmoke"></div>
    <div class="cinematic-icon" id="cinematicEventIcon">🏆</div>
    <div class="cinematic-title" id="cinematicEventTitle">Victory</div>
    <div class="cinematic-subtitle" id="cinematicEventSubtitle"></div>
  </div>
</div>
<div class="map-hud" id="mapHud">
  <button class="map-hud-drag-btn" id="mapHudDragBtn" title="Move HUD">⠿</button>
  <div class="map-hud-body" id="mapHudBody">
    <div class="map-hud-row">
      <button class="btn btn-sm active" id="mapJapanBtn" onclick="toggleMapMode('japan')">🗾日本</button>
      <button class="btn btn-sm" id="mapWorldBtn" onclick="toggleMapMode('world')">🌐世界</button>
      <button class="btn btn-sm" id="mapSpaceBtn" onclick="toggleMapMode('space')">🚀宇宙</button>
      <button class="btn btn-sm" id="mapCenterBtn" onclick="resetMapView()">中心</button>
      <button class="btn btn-sm" id="mapTraceBtn" onclick="toggleMapTracePanel()">📐座標</button>
      <button class="btn btn-sm" id="mapFocusModeBtn" onclick="toggleFocusMode()">🎯集中</button>
    </div>
    <div class="map-hud-row secondary">
      <span class="pill" id="mapHudMode">日本マップ</span>
      <span class="pill" id="mapHudFocus">注目: 日本</span>
      <span class="pill" id="mapHudZoom">Zoom x1.00</span>
    </div>
    <div class="map-hud-row tertiary" id="mapTraceRow">
      <div class="map-trace-panel">
        <div class="map-trace-toolbar">
          <span class="pill" id="mapTraceCursor">カーソル: --</span>
          <span class="pill" id="mapTracePointCount">0点</span>
          <button class="btn btn-sm" id="mapTraceActiveBtn" onclick="toggleMapTraceDrawing()">なぞりOFF</button>
          <button class="btn btn-sm" onclick="clearMapTrace()">消去</button>
          <button class="btn btn-sm btn-blue" onclick="copyMapTrace()">コピー</button>
        </div>
        <div class="map-trace-hint" id="mapTraceHint">座標ツールを開くと、左ドラッグで日本地図をなぞって緯度経度の点列を取れます。</div>
        <textarea class="map-trace-output" id="mapTraceOutput" readonly></textarea>
      </div>
    </div>
  </div>
  <button class="map-hud-fold-btn" id="mapHudFoldBtn" onclick="toggleMobileMapHud()">▴</button>
</div>
<div class="focus-tray focus-tray-left" id="focusTray">
  <button class="focus-tray-toggle" id="focusTrayToggle" onclick="toggleFocusTray()">▶</button>
  <div class="focus-tray-panel" id="focusTrayPanel"></div>
</div>
<div class="focus-time-dock" id="focusTimeDock"></div>
<button class="mobile-ui-toggle" id="mobileTopToggle" onclick="toggleMobileChrome('top')">▾</button>
<button class="mobile-ui-toggle" id="mobileBottomToggle" onclick="toggleMobileChrome('bottom')">▴</button>
<button class="mobile-ui-toggle" id="mobilePanelToggle" onclick="toggleMobileChrome('panel')">◀</button>

<!-- GAME AREA -->
<div id="gameArea"><canvas id="mapCanvas"></canvas></div>

<!-- RIGHT PANEL -->
<div id="rightPanel">
  <div id="panelTabs">
    <button class="btn active" data-panel="market">📊市場</button>
    <button class="btn" data-panel="news">📰ニュース</button>
    <button class="btn" data-panel="goals">🎯戦略</button>
    <button class="btn" data-panel="research">🧪研究</button>
    <button class="btn" data-panel="world">🌐世界</button>
    <button class="btn" data-panel="foreign">🌍海外株</button>
    <button class="btn" data-panel="compare">🌍比較</button>
    <button class="btn" data-panel="resources">📦資源</button>
    <button class="btn" data-panel="infra">🏗公共</button>
    <button class="btn" data-panel="trade">🚢貿易</button>
    <button class="btn" data-panel="develop">🏗開発</button>
    <button class="btn" data-panel="bank">🏦日銀</button>
    <button class="btn" data-panel="law">⚖法律</button>
    <button class="btn" data-panel="cyber">💻サイバー</button>
    <button class="btn" data-panel="chaos">🌋災害</button>
    <button class="btn" data-panel="space">🚀宇宙</button>
    <button class="btn" data-panel="nuke">🪖戦力</button>
  </div>
  <div id="panelContent"></div>
</div>

<div class="home-screen show" id="homeScreen">
  <div class="title-fun-layer" id="titleFunLayer">
    <button class="title-sign" data-tone="economy" style="left:7%;top:10%" onclick="dropTitleSign(this)">景気!</button>
    <button class="title-sign" data-tone="market" style="left:76%;top:12%" onclick="dropTitleSign(this)">株!</button>
    <button class="title-sign" data-tone="public" style="left:16%;top:78%" onclick="dropTitleSign(this)">公共!</button>
    <button class="title-sign" data-tone="space" style="left:83%;top:72%" onclick="dropTitleSign(this)">宇宙!</button>
  </div>
  <div class="home-ambient-layer" id="homeAmbientLayer" aria-hidden="true">
    <div class="home-ambient-tile tl"></div>
    <div class="home-ambient-tile tr"></div>
    <div class="home-ambient-tile bl"></div>
    <div class="home-ambient-tile br"></div>
    <div class="home-ambient-tile mid"></div>
  </div>
  <div class="home-card">
    <div class="home-title" id="homeTitleText">日本経済シミュレーター</div>
    <div class="home-sub" id="homeSubText">国家経営、株式市場、研究、外交、宇宙、核、重大イベントをまとめて動かす長期戦略シミュレーションです。ここからゲーム開始、前回のセーブ再開、設定、ワールド作成ができます。</div>
    <div class="home-title-tools">
      <button class="btn btn-sm" id="homeDropSignsBtn" onclick="dropAllTitleSigns()">看板を落とす</button>
      <button class="btn btn-sm" id="homeSettingsOpenBtn" onclick="openAction('settings')">設定</button>
      <button class="btn btn-sm" id="homeExitBtn" onclick="exitGame()">終了</button>
    </div>
    <div class="home-showcase">
      <div class="home-showcase-main">
        <div class="home-showcase-kicker" id="homeHeroKicker">National Command Deck</div>
        <div class="home-showcase-value" id="homeHeroValue">47都道府県 / 0企業 / 0か国</div>
        <div class="home-showcase-note" id="homeHeroNote">株式市場、外交、研究、災害、宇宙、戦争までを一つの国家運営でつなげて回す長期戦略シミュレーションです。</div>
        <div class="home-showcase-actions">
          <button class="home-main-btn" id="homeHeroStartBtn" onclick="startFromHome()">START</button>
          <button class="home-sub-btn" id="homeHeroContinueBtn" onclick="continueFromHome()">📂 前回のセーブ</button>
          <button class="home-sub-btn" id="homeHeroSaveBtn" onclick="titleOpenSaveList()">SAVE DATA</button>
          <button class="home-sub-btn" id="homeHeroResetBtn" onclick="resetFromHome()">NEW GAME</button>
          <button class="home-sub-btn" id="homeHeroTutorialBtn" onclick="startTutorialFromHome()">📘 TUTORIAL</button>
        </div>
        <div class="home-difficulty-strip">
          <div class="home-difficulty-head">
            <span class="label" id="homeDifficultyLabel">難易度</span>
            <span class="value" id="homeDifficultyValue">標準</span>
          </div>
          <div class="home-difficulty-actions">
            <button class="btn btn-sm btn-blue" id="homeWorldCreateBtn" onclick="beginWorldCreation()">🧭 ワールド作成</button>
          </div>
          <div class="home-difficulty-note" id="homeDifficultyNote">通常は標準難易度。ハードでは株価と売上の揺れが大きく、イベントショックも強くなります。</div>
        </div>
      </div>
      <div class="home-showcase-side">
        <div class="home-showcase-card">
          <span class="label" id="homeHeroSaveLabel">最新セーブ</span>
          <span class="value" id="homeHeroSaveValue">AUTO / 2025年 第1週</span>
          <span class="note" id="homeHeroSaveNote">前回のセーブからそのまま再開できます。</span>
        </div>
        <div class="home-showcase-card">
          <span class="label" id="homeHeroModeLabel">Mode</span>
          <span class="value" id="homeHeroModeValue">日本語 / PC / 通常</span>
          <span class="note" id="homeHeroModeNote">難易度と試験モードはワールド作成時に決まります。今のワールドでは固定です。</span>
        </div>
        <div class="home-showcase-card">
          <span class="label" id="homeHeroScaleLabel">Scope</span>
          <span class="value" id="homeHeroScaleValue">市場・外交・戦力・宇宙</span>
          <span class="note" id="homeHeroScaleNote">タイトル中央を広くして、現在の状況や規模が一目で分かるようにしています。</span>
        </div>
      </div>
    </div>
    <div class="home-grid">
      <div class="home-panel">
        <span class="home-badge" id="homeBadgeMain">ホーム</span>
        <div class="home-lang">
          <button class="btn btn-sm" id="langJaBtn" onclick="setLanguage('ja')">日本語</button>
          <button class="btn btn-sm" id="langEnBtn" onclick="setLanguage('en')">English</button>
          <button class="btn btn-sm" id="homeUiDesktopBtn" onclick="setUiMode('desktop')">PC</button>
          <button class="btn btn-sm" id="homeUiMobileBtn" onclick="setUiMode('mobile')">スマホ</button>
        </div>
        <div class="home-list" id="homeInfoList">
          <div>重大イベントは発生頻度を少し下げ、その分一回ごとの影響を大きくします。</div>
          <div>研究は対象企業を 100% 保有していないと実行できません。</div>
          <div>試験モードを切ると、支持率 0% で政権崩壊して最初からやり直しになります。</div>
        </div>
        <div class="home-save-list collapsed" id="homeSaveList"></div>
        <div class="home-message" id="homeMessage">ホーム画面から開始できます。</div>
      </div>
      <div class="home-panel">
        <span class="home-badge" id="homeBadgeSetting">設定</span>
        <div class="home-setting-row">
          <span id="homeSandboxLabel">試験モード</span>
          <span class="pill" id="homeSandboxValue">OFF</span>
        </div>
        <div class="home-setting-row">
          <span id="homeAutosaveLabel">オートセーブ</span>
          <button class="btn btn-sm" id="homeAutosaveBtn" onclick="toggleAutoSaveFromHome()">ON</button>
        </div>
        <div class="home-setting-row">
          <span id="homeCurrentSaveLabel">現在のオートセーブ</span>
          <span id="homeAutosaveMeta" class="compact-note">未作成</span>
        </div>
        <div class="home-setting-row">
          <span id="homePresetLabel">想定プレイ傾向</span>
          <span class="compact-note" id="homePresetValue">中長期 / 高難度 / 強い相場変動</span>
        </div>
        <div class="home-setting-row">
          <span id="homeDifficultySettingLabel">難易度</span>
          <div class="home-setting-col">
            <div class="save-actions">
              <button class="btn btn-sm btn-blue" id="homeSettingWorldCreateBtn" onclick="beginWorldCreation()">🧭 ワールド作成</button>
            </div>
            <span class="compact-note" id="homeDifficultySettingNote">標準</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

<!-- BOTTOM BAR -->
<div id="bottomBar">
  <button class="btn" id="bottomHomeBtn" onclick="openHomeScreen(S.language==='en' ? 'Returned to the title screen.' : 'タイトルに戻りました。')">🏠タイトル</button>
  <button class="btn btn-green" id="bottomBudgetBtn" onclick="openAction('budget')">💰年間予算</button>
  <button class="btn btn-blue" id="bottomSaveBtn" onclick="openAction('save')">💾保存</button>
  <button class="btn" id="bottomSettingsBtn" onclick="openAction('settings')">⚙設定</button>
  <button class="btn" id="bottomFocusBtn" onclick="toggleFocusMode()">🎯集中</button>
<div id="speedCtrl">
    <button class="btn btn-sm" id="timeRealtimeBtn" data-time-mode="realtime" onclick="setTimeMode('realtime')">⏱リアル</button>
    <button class="btn btn-sm" data-speed="0" onclick="setSpeed(0)">⏸</button>
    <button class="btn btn-sm active" data-speed="1" onclick="setSpeed(1)">▶</button>
    <button class="btn btn-sm" data-speed="2" onclick="setSpeed(2)">▶▶</button>
    <button class="btn btn-sm" data-speed="4" onclick="setSpeed(4)">▶▶▶</button>
    <span class="speed-divider"></span>
    <button class="btn btn-sm" id="timeTurnBtn" data-time-mode="turn" onclick="setTimeMode('turn')">🎲ターン</button>
    <button class="btn btn-sm" id="turnStepBtn" onclick="cycleTurnStep()">1週</button>
    <button class="btn btn-sm btn-yellow" id="turnAdvanceBtn" onclick="advanceTurn()">⏭進む</button>
  </div>
</div>

<!-- MODAL -->
<div class="modal-overlay" id="actionModal">
  <div class="modal">
    <span class="modal-close" onclick="closeModal()">✕</span>
    <h2 id="modalTitle"></h2>
    <div id="modalContent"></div>
  </div>
</div>

<div id="tooltip"></div>
<div class="view-loader" id="viewLoader">
  <div class="view-loader-card">
    <span class="view-loader-spinner"></span>
    <span class="view-loader-label" id="viewLoaderLabel">読込中...</span>
  </div>
</div>

<script>
// ============================================================
// GAME STATE
// ============================================================
const S = {
  money:90000, polCap:100, polCapMax:400, pop:12600, gdp:5400, approval:58,
  week:1, year:2025, speed:1, bubbleIndex:0,
  prevMoney:90000, prevPolCap:100, prevPop:12600, prevGdp:5400, prevApproval:58, prevTechLevel:12, prevYenValue:100,
  militaryBudget:5, educationBudget:5, techBudget:5, infraBudget:5,
  taxRate:18,
  budgetDraft:{military:12000, education:9000, technology:11000, company:'', companySupport:0, companySupports:[{company:'', amount:0},{company:'', amount:0}]},
  annualBudget:{military:12000, education:9000, technology:11000, company:'', companySupport:0, companySupports:[{company:'', amount:0},{company:'', amount:0}]},
  budgetSetYear:2024,
  tradeDeals:[], tradeShipments:[], loans:[{amount:10000000, rate:0.0009, original:10000000, termWeeks:166666, weeklyPrincipal:60, seeded:true}], ownedStocks:{}, foreignOwnedStocks:{},
  stockCostBasis:{},
  bubbleActive:false, bubbleSector:null, bubblePeak:0, nextBubbleEligibleWeek:260,
  trackedPrices:['copper','oil','gold','lithium','silicon'],
  currentPanel:'market', companySortMode:'price', companySectorFilter:'all',
  historyMetric:'gdp', historyRange:260, historyMode:'macro',
  defPower:10, cyberPower:5, nukePower:0, nuclearPlants:0, energyBonus:0,
  techLevel:12, technologyMomentum:0, yenValue:100, businessCycle:0.08, businessCyclePhase:0,
  cyberInvest:0, infraLevel:0,
  sandboxMode:false,
  difficultyMode:'normal',
  mapOffsetX:0, mapOffsetY:0, mapZoom:1, mapDragging:false, mapDragStart:{x:0,y:0},
  currentMapMode:'japan', mapHoverTarget:null, mapFocus:'panel',
  japanView:{offsetX:0, offsetY:0, zoom:1},
  worldView:{offsetX:0, offsetY:0, zoom:1},
  spaceView:{offsetX:-0.02, offsetY:-0.03, zoom:1.02},
  worldFocusCountry:'japan',
  japanFocusPrefecture:'tokyo',
  pendingActions:[], // queue for paused state
  discoveredMaterials:[], // unique materials from dev
  publicWorks:[], draggingWork:null, buildRepeatMode:true, draggingStrike:null, strikeRepeatMode:true, draggingDisaster:null,
  transportNetworks:[], draggingRoute:null, transportRepeatMode:true,
  energy:{production:0, demand:0},
  totalWeeks:0,
  foreignUnrest:0, // global economic worry → japan stock boost
  activeMajorEvent:null, majorEventCooldown:0,
  companyChartRange:260, companyChartMode:'price',
  spaceRouteProgress:0, spaceFocusBody:'moon',
  tradeQuantityDraft:5,
  tradeDraftMode:'percent',
  tradeAmountDraft:1000,
  tradeFocusResource:'',
  subsidyHeat:{},
  researchProjects:[], completedResearch:[], unlockedProducts:[],
  started:false, homeMessage:'', bioWeaponUnlocks:[],
  language:'ja',
  warVictories:[], activeWar:null, foreignCompanies:[],
  spyMissions:[],
  militaryInventory:{infantry:0,tanks:0,fighters:0,ships:0,drones:0,spies:0},
  militaryProjects:[],
  mapEffects:[],
  lastDividendYear:2024,
  saveLabels:{},
  lastUsedSaveSlot:'autosave',
  homeSaveExpanded:false,
  socialUnrest:18,
  birthRate:7.4,
  bankLoanDraftCho:1000,
  briberyExposure:0,
  ideologyScore:28,
  ideologyDrift:0,
  lawChangeIntervalYears:3,
  nextLawChangeWeek:0,
  lastLawChangeId:'',
  chaosRepeatMode:false,
  timeMode:'realtime',
  realtimeSpeed:1,
  turnStepWeeks:1,
  turnProcessing:false,
  uiMode:'desktop',
  mobileTopCollapsed:false,
  mobileBottomCollapsed:false,
  mobilePanelCollapsed:false,
  mobileMapHudCollapsed:false,
  mapTraceOpen:false,
  mapTraceActive:false,
  mapTraceDrawing:false,
  mapTracePoints:[],
  prefectureEnhancementBrush:null,
  prefectureBrushSelection:null,
  focusMode:false,
  focusTrayOpen:false,
  focusTrayPosition:'left',
  focusTrayStats:['money','gdp','approval','yen'],
  qualityMode:'balanced',
  worldConfigLocked:true,
  newsDockCollapsed:false,
  historyDockCollapsed:true,
  newsDockPos:{left:10, top:116},
  historyDockPos:{left:362, top:116},
  mapHudOffset:{x:0,y:0},
  infraMapFilter:'all',
  selectedChaosCompany:'',
  forcedMigrationDraft:{from:'tokyo', to:'osaka', amount:18},
  weeklyBreakdown:{tax:0,budget:0,trade:0,dividend:0,space:0,interest:0,principal:0,companySupport:0,publicRevenue:0,operations:0,socialSecurity:0,healthcare:0,childcare:0,administration:0,adjustments:0},
};

const DEFAULT_STATE = Object.freeze({
  money:S.money,
  polCap:S.polCap,
  gdp:S.gdp,
  approval:S.approval,
  techLevel:S.techLevel,
  yenValue:S.yenValue,
  bubbleIndex:S.bubbleIndex,
  birthRate:S.birthRate,
});

const STORAGE_PREFIX = 'jp-econ-sim-v4';
const SAVE_SLOTS = ['autosave','slot1','slot2','slot3','slot4','slot5','slot6','slot7','slot8'];
const LAST_RESUME_SLOT_KEY = `${STORAGE_PREFIX}:last-resume-slot`;
const LANGUAGE_PREF_KEY = `${STORAGE_PREFIX}:language`;
const INSTALLATION_ID_KEY = `${STORAGE_PREFIX}:installation-id`;
const ACTIVATION_INFO_KEY = `${STORAGE_PREFIX}:activation-info`;
const SECURITY_STATE_KEY = `${STORAGE_PREFIX}:security-state`;
let INITIAL_SNAPSHOT = null;

function getStoredLanguagePreference(){
  try{
    const lang = localStorage.getItem(LANGUAGE_PREF_KEY);
    return ['ja','en'].includes(lang) ? lang : '';
  } catch(error){
    return '';
  }
}

function persistLanguagePreference(lang){
  try{
    if(['ja','en'].includes(lang)) localStorage.setItem(LANGUAGE_PREF_KEY, lang);
  } catch(error){}
}

function getPreferredLanguage(fallback='ja'){
  return getStoredLanguagePreference() || fallback || 'ja';
}

function createInstallationId(){
  return `inst-${Date.now().toString(36)}-${Math.random().toString(36).slice(2,10)}`;
}
function getInstallationId(){
  try{
    let id = localStorage.getItem(INSTALLATION_ID_KEY);
    if(!id){
      id = createInstallationId();
      localStorage.setItem(INSTALLATION_ID_KEY, id);
    }
    return id;
  } catch(error){
    if(!window.__fallbackInstallationId) window.__fallbackInstallationId = createInstallationId();
    return window.__fallbackInstallationId;
  }
}
function getActivationInfo(){
  try{
    const raw = localStorage.getItem(ACTIVATION_INFO_KEY);
    if(!raw) return {status:'trial', key:'', activatedAt:'', installationId:getInstallationId()};
    const parsed = JSON.parse(raw);
    return {
      status:parsed?.status === 'active' ? 'active' : 'trial',
      key:parsed?.key || '',
      activatedAt:parsed?.activatedAt || '',
      installationId:parsed?.installationId || getInstallationId(),
    };
  } catch(error){
    return {status:'trial', key:'', activatedAt:'', installationId:getInstallationId()};
  }
}
function persistActivationInfo(info){
  try{
    localStorage.setItem(ACTIVATION_INFO_KEY, JSON.stringify({
      status:info?.status === 'active' ? 'active' : 'trial',
      key:info?.key || '',
      activatedAt:info?.activatedAt || '',
      installationId:info?.installationId || getInstallationId(),
    }));
  } catch(error){}
}
function normalizeActivationKey(key){
  return String(key || '')
    .toUpperCase()
    .replace(/[^A-Z0-9]/g, '')
    .match(/.{1,4}/g)?.join('-') || '';
}
function simpleHashHex(value){
  const str = String(value || '');
  let h1 = 0x811c9dc5;
  let h2 = 0x01000193;
  for(let i = 0; i < str.length; i++){
    const code = str.charCodeAt(i);
    h1 ^= code;
    h1 = Math.imul(h1, 0x01000193);
    h2 ^= code + i;
    h2 = Math.imul(h2, 0x45d9f3b);
  }
  return `${(h1 >>> 0).toString(16).padStart(8,'0')}${(h2 >>> 0).toString(16).padStart(8,'0')}`;
}
function xorCipher(value, seed){
  const source = String(value || '');
  const key = String(seed || 'jp-econ-sim');
  let out = '';
  for(let i = 0; i < source.length; i++){
    out += String.fromCharCode(source.charCodeAt(i) ^ key.charCodeAt(i % key.length));
  }
  return out;
}
function getRuntimeFingerprint(){
  const nav = typeof navigator !== 'undefined' ? navigator : {};
  const screenData = typeof screen !== 'undefined' ? screen : {};
  return simpleHashHex([
    nav.userAgent || '',
    nav.language || '',
    nav.platform || '',
    screenData.width || 0,
    screenData.height || 0,
    Intl.DateTimeFormat().resolvedOptions().timeZone || '',
  ].join('|'));
}
function getSaveProtectionSeed(){
  return [
    STORAGE_PREFIX,
    getInstallationId(),
    'SEC2',
  ].join('|');
}
function readSecurityState(){
  try{
    const raw = localStorage.getItem(SECURITY_STATE_KEY);
    const parsed = raw ? JSON.parse(raw) : {};
    return {
      tamperDetected:!!parsed?.tamperDetected,
      lastAlert:parsed?.lastAlert || '',
      integrityBaseline:parsed?.integrityBaseline || '',
      suspiciousLoads:Math.max(0, parsed?.suspiciousLoads || 0),
    };
  } catch(error){
    return {tamperDetected:false, lastAlert:'', integrityBaseline:'', suspiciousLoads:0};
  }
}
function persistSecurityState(nextState){
  try{
    localStorage.setItem(SECURITY_STATE_KEY, JSON.stringify({
      tamperDetected:!!nextState?.tamperDetected,
      lastAlert:nextState?.lastAlert || '',
      integrityBaseline:nextState?.integrityBaseline || '',
      suspiciousLoads:Math.max(0, nextState?.suspiciousLoads || 0),
    }));
  } catch(error){}
}
function encodeUtf8Base64(value){
  return btoa(unescape(encodeURIComponent(String(value || ''))));
}
function decodeUtf8Base64(value){
  return decodeURIComponent(escape(atob(String(value || ''))));
}
function getActivationSeed(installationId=getInstallationId()){
  return simpleHashHex(`${STORAGE_PREFIX}|${installationId}|license`);
}
function getExpectedActivationKey(installationId=getInstallationId()){
  const seed = getActivationSeed(installationId);
  return normalizeActivationKey(`JES4${seed.slice(0, 12)}`);
}
function isActivationKeyValid(key, installationId=getInstallationId()){
  return normalizeActivationKey(key) === getExpectedActivationKey(installationId);
}
function registerSecurityAlert(message, type='bad'){
  if(!window.__securityAlerts) window.__securityAlerts = new Set();
  if(window.__securityAlerts.has(message)) return;
  window.__securityAlerts.add(message);
  try{ addEvent?.(`🔐 ${message}`, type); } catch(error){}
  console.warn('[security]', message);
}
function encodeProtectedSaveEnvelope(payload, slot=''){
  const installationId = getInstallationId();
  const savedAt = payload?.savedAt || new Date().toISOString();
  const plain = JSON.stringify(payload);
  const cipher = encodeUtf8Base64(xorCipher(plain, getSaveProtectionSeed()));
  const signature = simpleHashHex([STORAGE_PREFIX, slot, savedAt, installationId, plain].join('|'));
  return JSON.stringify({
    format:'SEC2',
    version:payload?.version || 0,
    savedAt,
    meta:payload?.meta || null,
    installationId,
    fingerprint:getRuntimeFingerprint(),
    cipher,
    signature,
  });
}
function decodeProtectedSaveEnvelope(raw, slot=''){
  const parsed = JSON.parse(raw);
  if(!parsed?.format || parsed.format !== 'SEC2'){
    return {
      payload:parsed,
      meta:parsed?.meta || null,
      valid:true,
      protected:false,
      installationMismatch:false,
      fingerprintMismatch:false,
    };
  }
  const installationMismatch = !!parsed.installationId && parsed.installationId !== getInstallationId();
  const fingerprintMismatch = !!parsed.fingerprint && parsed.fingerprint !== getRuntimeFingerprint();
  if(installationMismatch){
    return {
      payload:null,
      meta:parsed?.meta || null,
      valid:false,
      protected:true,
      installationMismatch:true,
      fingerprintMismatch,
    };
  }
  const plain = xorCipher(decodeUtf8Base64(parsed.cipher || ''), getSaveProtectionSeed());
  const expected = simpleHashHex([STORAGE_PREFIX, slot, parsed.savedAt || '', parsed.installationId || '', plain].join('|'));
  return {
    payload:JSON.parse(plain),
    meta:parsed?.meta || null,
    valid:expected === parsed.signature,
    protected:true,
    installationMismatch:false,
    fingerprintMismatch,
  };
}
function getStoredSaveRecord(slot){
  const raw = localStorage.getItem(getSaveStorageKey(slot));
  if(!raw) return null;
  const decoded = decodeProtectedSaveEnvelope(raw, slot);
  if(!decoded.valid || decoded.installationMismatch){
    const security = readSecurityState();
    security.tamperDetected = true;
    security.suspiciousLoads = (security.suspiciousLoads || 0) + 1;
    security.lastAlert = `save:${slot}:${decoded.installationMismatch ? 'install-mismatch' : 'sig-mismatch'}`;
    persistSecurityState(security);
    throw new Error(decoded.installationMismatch
      ? 'This save is bound to another installation.'
      : 'Protected save integrity check failed.');
  }
  if(decoded.fingerprintMismatch){
    registerSecurityAlert(
      currentI18n?.().loader
        ? (getPreferredLanguage() === 'en'
          ? 'Runtime fingerprint changed. Save was loaded in compatibility mode.'
          : '実行環境が変化したため、互換モードでセーブを読み込みました。')
        : 'Runtime fingerprint changed.',
      ''
    );
  }
  return decoded;
}
function readStoredSavePayload(slot){
  const record = getStoredSaveRecord(slot);
  return record?.payload || null;
}
function writeStoredSavePayload(slot, payload){
  localStorage.setItem(getSaveStorageKey(slot), encodeProtectedSaveEnvelope(payload, slot));
}
function getCriticalRuntimeSignature(){
  const targets = [
    ensureStateShape,
    createSaveData,
    applySaveData,
    getSaveMeta,
    performSave,
    window.loadGame,
    renderPanel,
    drawMap,
    gameTick,
  ].filter(fn => typeof fn === 'function');
  return simpleHashHex(targets.map(fn => String(fn)).join('\n@@\n'));
}
function refreshIntegrityBaseline(force=false){
  const security = readSecurityState();
  const current = getCriticalRuntimeSignature();
  if(force || !security.integrityBaseline){
    security.integrityBaseline = current;
    persistSecurityState(security);
  }
  return current;
}
function verifyRuntimeIntegrity(stage='runtime'){
  try{
    const current = getCriticalRuntimeSignature();
    const security = readSecurityState();
    if(!security.integrityBaseline){
      security.integrityBaseline = current;
      persistSecurityState(security);
      return true;
    }
    if(security.integrityBaseline !== current){
      security.tamperDetected = true;
      security.lastAlert = `${stage}:${new Date().toISOString()}`;
      persistSecurityState(security);
      registerSecurityAlert(
        getPreferredLanguage() === 'en'
          ? 'Runtime integrity mismatch detected. Some protections may be degraded.'
          : '実行時整合性の不一致を検知しました。保護機能の一部を制限します。'
      );
      return false;
    }
    return true;
  } catch(error){
    console.warn('[security] verifyRuntimeIntegrity failed', error);
    return false;
  }
}
function auditEconomyIntegrity(stage='runtime'){
  const defaults = {
    money:DEFAULT_STATE.money,
    polCap:DEFAULT_STATE.polCap,
    gdp:DEFAULT_STATE.gdp,
    approval:DEFAULT_STATE.approval,
    techLevel:DEFAULT_STATE.techLevel,
    yenValue:DEFAULT_STATE.yenValue,
    bubbleIndex:DEFAULT_STATE.bubbleIndex,
  };
  const repaired = [];
  Object.entries(defaults).forEach(([key, fallback]) => {
    if(!Number.isFinite(S[key])){
      S[key] = fallback;
      repaired.push(key);
    }
  });
  if(!S.sandboxMode && Number.isFinite(S.polCap) && Number.isFinite(S.polCapMax) && S.polCap > S.polCapMax){
    S.polCap = S.polCapMax;
    repaired.push('polCapClamp');
  }
  companies.forEach(company => {
    if(!Number.isFinite(company.price) || company.price < 100){
      company.price = Math.max(100, Number(company.base) || 100);
      repaired.push(`price:${company.name}`);
    }
    if(!Number.isFinite(company.revenue) || company.revenue < 0){
      company.revenue = Math.max(0, Number(company.baseRevenue) || Number(company.revenue) || 0);
      repaired.push(`revenue:${company.name}`);
    }
  });
  if(repaired.length){
    const security = readSecurityState();
    security.tamperDetected = true;
    security.lastAlert = `${stage}:${repaired.join(',').slice(0, 180)}`;
    persistSecurityState(security);
    registerSecurityAlert(
      getPreferredLanguage() === 'en'
        ? 'Suspicious runtime values were corrected automatically.'
        : '不正または破損の疑いがある値を自動補正しました。'
    );
  }
}
function deepFreeze(value, seen=new WeakSet()){
  if(!value || typeof value !== 'object' || seen.has(value)) return value;
  seen.add(value);
  Object.getOwnPropertyNames(value).forEach(key => deepFreeze(value[key], seen));
  return Object.freeze(value);
}
function finalizeStaticSecurity(){
  const freezeTargets = [
    typeof productCatalog !== 'undefined' ? productCatalog : null,
    typeof publicWorksCatalog !== 'undefined' ? publicWorksCatalog : null,
    typeof transportNetworkCatalog !== 'undefined' ? transportNetworkCatalog : null,
    typeof researchCatalog !== 'undefined' ? researchCatalog : null,
    typeof pathogenCatalog !== 'undefined' ? pathogenCatalog : null,
    typeof spaceBodies !== 'undefined' ? spaceBodies : null,
    typeof sectors !== 'undefined' ? sectors : null,
  ];
  freezeTargets.forEach(target => {
    try{ if(target) deepFreeze(target); } catch(error){}
  });
}

const I18N = {
  ja:{
    appTitle:'日本経済シミュレーター v4',
    homeTitle:'日本経済シミュレーター',
    homeSub:'国家経営、株式市場、研究、外交、宇宙、核、重大イベントをまとめて動かす長期戦略シミュレーションです。ここからゲーム開始、前回のセーブ再開、設定、ワールド作成ができます。',
    badgeMain:'ホーム',
    badgeSetting:'設定',
    start:'▶ ゲーム開始',
    continue:'📂 前回のセーブから開始',
    save:'💾 セーブ管理',
    reset:'↺ 最初から',
    sandbox:'試験モード',
    autosave:'オートセーブ',
    currentSave:'現在のオートセーブ',
    preset:'想定プレイ傾向',
    presetValue:'中長期 / 高難度 / 強い相場変動',
    info:[
      '重大イベントは発生頻度を少し下げ、その分一回ごとの影響を大きくします。',
      '研究は対象企業を 100% 保有していないと実行できません。',
      '試験モードを切ると、支持率 0% で政権崩壊して最初からやり直しになります。'
    ],
    noSave:'保存データはまだありません。',
    load:'開始',
    homeToggleSaveOpen:'💾 セーブ一覧を開く',
    homeToggleSaveClose:'📁 セーブ一覧を閉じる',
    homeContinueMissing:'📂 前回のセーブは未作成',
    homeContinuePrefix:'📂 前回のセーブ:',
    homeAutosaveEmpty:'未作成',
    homeTools:{drop:'看板を落とす', settings:'設定', exit:'終了'},
    stats:{
      money:'💰 国庫', polCap:'🏛 政治資本', pop:'👥 人口', gdp:'📈 GDP',
      approval:'😊 支持率', bubble:'🎈 バブル', yen:'💱 円指数', tech:'🧠 技術力',
      defense:'🛡 防衛力', cyber:'💻 サイバー', nuclear:'☢ 核', sandbox:'🧪 試験'
    },
    map:{
      japanBtn:'🗾日本', worldBtn:'🌐世界', spaceBtn:'🚀宇宙', centerBtn:'中心',
      focusModeBtn:'🎯集中', japan:'日本マップ', world:'世界マップ', space:'宇宙マップ', mapGeneric:'マップ',
      focus:'注目', focusJapan:'注目: 日本列島', focusSpace:'注目: 宇宙', placing:'配置中'
    },
    bottom:{home:'🏠タイトル', budget:'💰年間予算', save:'💾保存', settings:'⚙設定', focus:'🎯集中'},
    bottomRealtime:'⏱リアル',
    bottomTurn:'🎲ターン',
    bottomAdvance:'⏭進む',
    tabs:{
      market:'📊市場', news:'📰ニュース', goals:'🎯戦略', research:'🧪研究', world:'🌐世界',
      foreign:'🌍海外株', compare:'🌍比較', resources:'📦資源', infra:'🏗公共', trade:'🚢貿易',
      develop:'🏗開発', bank:'🏦日銀', law:'⚖法律', cyber:'💻サイバー', chaos:'🌋災害',
      space:'🚀宇宙', nuke:'🪖戦力'
    },
    saveManager:{
      title:'💾 セーブ管理', desc:'13週ごとにオートセーブします。手動スロットは長期プレイや分岐用です。',
      empty:'まだ保存されていません。', save:'保存', load:'読込', rename:'名前変更', delete:'削除',
      export:'JSON書き出し', normalOn:'試験モード ON', normalOff:'試験モード OFF',
      note:'試験モード中は開始資金と政治資本を大きく確保します。OFF にすると以後は通常運営に戻ります。',
      slotEmpty:'空', yearWeek:'年週', treasury:'国庫', approval:'支持率', score:'目標達成'
    },
    settings:{
      title:'⚙ 設定', desc:'タイトル中でもプレイ中でも、言語や自動保存などの基本設定をすぐ変えられます。',
      language:'言語', languageNote:'主要UI・マップHUD・タイトルを即時切り替えます。',
      autosave:'オートセーブ', autosaveOn:'13週ごとに自動保存します。', autosaveOff:'自動保存は停止中です。',
      sandbox:'試験モード', sandboxOn:'高資金・高政治資本で実験できます。', sandboxOff:'通常ルールで運営します。',
      current:'現在のオートセーブ', close:'閉じる',
      focusMode:'集中モード', focusModeOn:'ブラウザが許可すれば全画面表示に入り、上部のUIをたたみます。', focusModeOff:'通常表示へ戻します。',
      uiMode:'画面モード', uiDesktop:'PC', uiMobile:'スマホ', uiMobileNote:'スマホモードでは上・下・右パネルを折りたたみできます。',
      mobileTop:'上バー', mobileBottom:'下バー', mobilePanel:'右パネル', opened:'開', closed:'閉',
      timeMode:'進行モード', realtime:'自動進行', turn:'ターン制',
      timeRealtimeNote:'時間が自動で流れます。高性能端末向けです。',
      timeTurnNote:'「進む」を押した時だけ時間を進めます。低スペック端末向けです。',
      turnStep:'1回で進む期間', turnLoader:'ターンを進行中...',
      turnStep1:'1週', turnStep4:'4週', turnStep13:'13週'
    },
    loader:'読込中...',
    units:{oku:'億', man:'万', cho:'兆', perWeek:'/週'},
    weekFormat:(year, week) => `${year}年 第${week}週`,
    titleSigns:['景気!','株!','公共!','宇宙!']
  },
  en:{
    appTitle:'Japan Economy Simulator v4',
    homeTitle:'Japan Economy Simulator',
    homeSub:'A long-form strategy simulation where you run national policy, stock markets, research, diplomacy, space, nuclear projects, and major crises. Start playing, resume your latest save, open settings, or create a new world from here.',
    badgeMain:'Home',
    badgeSetting:'Settings',
    start:'▶ Start Game',
    continue:'📂 Start From Latest Save',
    save:'💾 Save Manager',
    reset:'↺ New Game',
    sandbox:'Sandbox Mode',
    autosave:'Auto Save',
    currentSave:'Current Autosave',
    preset:'Play Style',
    presetValue:'Long-term / High difficulty / Strong market swings',
    info:[
      'Major events are less frequent, but each one hits much harder.',
      'Research requires 100% ownership of the partner company.',
      'With sandbox off, approval reaching 0% causes game over and a restart.'
    ],
    noSave:'No save data yet.',
    load:'Start',
    homeToggleSaveOpen:'💾 Open Save List',
    homeToggleSaveClose:'📁 Close Save List',
    homeContinueMissing:'📂 No recent save yet',
    homeContinuePrefix:'📂 Continue:',
    homeAutosaveEmpty:'None yet',
    homeTools:{drop:'Drop Signs', settings:'Settings', exit:'Exit'},
    stats:{
      money:'💰 Treasury', polCap:'🏛 Political Capital', pop:'👥 Population', gdp:'📈 GDP',
      approval:'😊 Approval', bubble:'🎈 Bubble', yen:'💱 Yen Index', tech:'🧠 Tech',
      defense:'🛡 Defense', cyber:'💻 Cyber', nuclear:'☢ Nuclear', sandbox:'🧪 Sandbox'
    },
    map:{
      japanBtn:'🗾Japan', worldBtn:'🌐World', spaceBtn:'🚀Space', centerBtn:'Center',
      focusModeBtn:'🎯Focus', japan:'Japan Map', world:'World Map', space:'Space Map', mapGeneric:'Map',
      focus:'Focus', focusJapan:'Focus: Japan', focusSpace:'Focus: Space', placing:'Placing'
    },
    bottom:{home:'🏠Title', budget:'💰Budget', save:'💾Save', settings:'⚙Settings', focus:'🎯Focus'},
    bottomRealtime:'⏱Realtime',
    bottomTurn:'🎲Turn',
    bottomAdvance:'⏭Advance',
    tabs:{
      market:'📊Market', news:'📰News', goals:'🎯Goals', research:'🧪Research', world:'🌐World',
      foreign:'🌍Overseas', compare:'🌍Compare', resources:'📦Resources', infra:'🏗Public Works', trade:'🚢Trade',
      develop:'🏗Develop', bank:'🏦Central Bank', law:'⚖Laws', cyber:'💻Cyber', chaos:'🌋Disasters',
      space:'🚀Space', nuke:'🪖Forces'
    },
    saveManager:{
      title:'💾 Save Manager', desc:'Autosave runs every 13 weeks. Manual slots are great for long campaigns and branching runs.',
      empty:'No save data in this slot yet.', save:'Save', load:'Load', rename:'Rename', delete:'Delete',
      export:'Export JSON', normalOn:'Sandbox ON', normalOff:'Sandbox OFF',
      note:'Sandbox keeps your treasury and political capital high for testing. Turning it off returns to normal rules.',
      slotEmpty:'Empty', yearWeek:'Date', treasury:'Treasury', approval:'Approval', score:'Goals'
    },
    settings:{
      title:'⚙ Settings', desc:'Change language, autosave, and sandbox options instantly from the title screen or while playing.',
      language:'Language', languageNote:'Switches the main UI, map HUD, and title screen immediately.',
      autosave:'Autosave', autosaveOn:'Automatically saves every 13 weeks.', autosaveOff:'Autosave is currently disabled.',
      sandbox:'Sandbox Mode', sandboxOn:'Experiment with very high money and political capital.', sandboxOff:'Play under the normal rule set.',
      current:'Current Autosave', close:'Close',
      focusMode:'Focus Mode', focusModeOn:'Enters fullscreen when the browser allows it and hides the top chrome for a cleaner view.', focusModeOff:'Returns to the normal layout.',
      uiMode:'Screen Mode', uiDesktop:'Desktop', uiMobile:'Mobile', uiMobileNote:'Mobile mode lets you manually collapse the top bar, bottom bar, and right panel.',
      mobileTop:'Top Bar', mobileBottom:'Bottom Bar', mobilePanel:'Right Panel', opened:'Open', closed:'Closed',
      timeMode:'Time Flow', realtime:'Real-time', turn:'Turn-based',
      timeRealtimeNote:'Time advances automatically. Best for high-performance devices.',
      timeTurnNote:'Time advances only when you press Advance. Best for low-spec PCs and phones.',
      turnStep:'Advance Size', turnLoader:'Processing turn...',
      turnStep1:'1 week', turnStep4:'4 weeks', turnStep13:'13 weeks'
    },
    loader:'Loading...',
    units:{oku:'B', man:'M', cho:'T', perWeek:'/wk'},
    weekFormat:(year, week) => `Week ${week}, ${year}`,
    titleSigns:['Growth!','Stocks!','Works!','Space!']
  }
};

const PANEL_LABEL_ORDER = ['market','news','goals','research','world','foreign','compare','resources','infra','trade','develop','bank','law','cyber','chaos','space','nuke'];

function currentI18n(){
  return I18N[S.language] || I18N.ja;
}

const RESEARCH_IMAGE_MAP = Object.freeze({
  antiviral_platform:'assets/research-art/research-antiviral-platform.png',
  orbital_fabrication:'assets/research-art/research-orbital-fabrication.png',
  fusion_grid:'assets/research-art/research-fusion-grid.png',
  quantum_network:'assets/research-art/research-quantum-network.png',
  europa_biotech:'assets/research-art/research-europa-biotech.png',
  smart_mobility:'assets/research-art/research-smart-mobility.png',
  relief_biologics:'assets/research-art/research-relief-biologics.png',
  climate_shield:'assets/research-art/research-climate-shield.png',
  urban_resilience:'assets/research-art/research-urban-resilience.png',
  amber_fever:'assets/research-art/research-amber-fever.png',
  velvet_strain:'assets/research-art/research-velvet-strain.png',
});

function getResearchImagePath(projectId){
  return RESEARCH_IMAGE_MAP[projectId] || '';
}

function renderResearchHero(project, kind='standard'){
  const src = getResearchImagePath(project.id);
  if(!src) return '';
  const badge = kind === 'pathogen'
    ? (S.language === 'en' ? 'BIOHAZARD PROGRAM' : '特殊病原体計画')
    : (S.language === 'en' ? 'NATIONAL RESEARCH' : '国家研究プロジェクト');
  return `<div class="research-hero">
    <img src="${src}" alt="${escapeHtml(project.name)}" loading="lazy" decoding="async">
    <div class="research-hero-badge ${kind === 'pathogen' ? 'pathogen' : ''}">${badge}</div>
  </div>`;
}

function formatWeekText(year=S.year, week=S.week){
  return currentI18n().weekFormat(year, week);
}

function getDefaultSaveLabel(slot){
  const t = currentI18n();
  return slot === 'autosave' ? (S.language === 'en' ? 'Continue' : '前回の続き') : (S.language === 'en' ? `Save ${slot.replace('slot','')}` : `セーブ ${slot.replace('slot','')}`);
}

function getPanelLabel(panel){
  return currentI18n().tabs?.[panel] || panel;
}

function buildPanelTabsHtml(){
  return PANEL_LABEL_ORDER.map(panel => `<button class="btn ${S.currentPanel===panel?'active':''}" data-panel="${panel}">${getPanelLabel(panel)}</button>`).join('');
}

function updateChromeLanguage(){
  const t = currentI18n();
  document.documentElement.lang = S.language === 'en' ? 'en' : 'ja';
  document.title = t.appTitle;
  const statMap = t.stats || {};
  document.querySelectorAll('#topBar .stat[data-stat]').forEach(stat => {
    const label = stat.querySelector('.lbl');
    if(label && statMap[stat.dataset.stat]) label.textContent = statMap[stat.dataset.stat];
  });
  const sandboxLabel = document.querySelector('#topBar .stat:not([data-stat]) .lbl');
  if(sandboxLabel) sandboxLabel.textContent = statMap.sandbox || sandboxLabel.textContent;
  const mapTexts = t.map || {};
  const mapIds = {
    mapJapanBtn:mapTexts.japanBtn,
    mapWorldBtn:mapTexts.worldBtn,
    mapSpaceBtn:mapTexts.spaceBtn,
    mapCenterBtn:mapTexts.centerBtn,
    mapFocusModeBtn:mapTexts.focusModeBtn || (S.language === 'en' ? '🎯Focus' : '🎯集中'),
  };
  Object.entries(mapIds).forEach(([id, value]) => {
    const el = document.getElementById(id);
    if(el && value) el.textContent = value;
  });
  const bottomTexts = t.bottom || {};
  const bottomIds = {
    bottomHomeBtn:bottomTexts.home,
    bottomBudgetBtn:bottomTexts.budget,
    bottomSaveBtn:bottomTexts.save,
    bottomSettingsBtn:bottomTexts.settings,
    bottomFocusBtn:bottomTexts.focus || (S.language === 'en' ? '🎯Focus' : '🎯集中'),
    timeRealtimeBtn:t.bottomRealtime,
    timeTurnBtn:t.bottomTurn,
    turnAdvanceBtn:t.bottomAdvance,
  };
  Object.entries(bottomIds).forEach(([id, value]) => {
    const el = document.getElementById(id);
    if(el && value) el.textContent = value;
  });
  const toolTexts = t.homeTools || {};
  const toolIds = {
    homeDropSignsBtn:toolTexts.drop,
    homeSettingsOpenBtn:toolTexts.settings,
    homeExitBtn:toolTexts.exit,
  };
  Object.entries(toolIds).forEach(([id, value]) => {
    const el = document.getElementById(id);
    if(el && value) el.textContent = value;
  });
  document.querySelectorAll('#titleFunLayer .title-sign').forEach((sign, index) => {
    const label = t.titleSigns?.[index];
    if(label) sign.textContent = label;
  });
  document.querySelectorAll('#panelTabs [data-panel]').forEach(button => {
    button.textContent = getPanelLabel(button.dataset.panel);
  });
  const loaderLabel = document.getElementById('viewLoaderLabel');
  if(loaderLabel && !document.getElementById('viewLoader')?.classList.contains('show')) loaderLabel.textContent = t.loader;
  const homeUiDesktopBtn = document.getElementById('homeUiDesktopBtn');
  const homeUiMobileBtn = document.getElementById('homeUiMobileBtn');
  if(homeUiDesktopBtn) homeUiDesktopBtn.textContent = t.settings?.uiDesktop || (S.language === 'en' ? 'Desktop' : 'PC');
  if(homeUiMobileBtn) homeUiMobileBtn.textContent = t.settings?.uiMobile || (S.language === 'en' ? 'Mobile' : 'スマホ');
  const newsDockTitle = document.getElementById('newsDockTitle');
  if(newsDockTitle) newsDockTitle.textContent = S.language === 'en' ? '📰 News' : '📰 ニュース';
  updateMapHud();
}

function clamp(v, min, max){ return Math.max(min, Math.min(max, v)); }
function rand(min, max){ return min + Math.random() * (max - min); }
function pick(arr){ return arr[Math.floor(Math.random() * arr.length)]; }
function lerp(a,b,t){ return a + (b-a) * t; }

function createEmptyHistory(){
  return {
    weeks:[], money:[], gdp:[], approval:[], bubble:[], pop:[],
    debt:[], inflation:[], unemployment:[], stability:[], def:[], cyber:[], tech:[], yen:[], cycle:[], unrest:[], ideology:[], polCap:[], nuclear:[]
  };
}

function createSpaceState(){
  return {level:0, rockets:0, rocketLevel:0, successes:0, failures:0, resources:0, factories:0, orbitalRelays:0, mission:null, discoveries:[], routeTrail:[], solarUnlocked:false, planets:{}};
}

function createGoalState(){
  return {
    economicPower:false,
    techSupremacy:false,
    energySecurity:false,
    resourceFrontier:false,
    publicTrust:false,
    spaceAge:false,
    cyberMastery:false,
    sovereignIndustry:false,
  };
}

function createNuclearState(){
  return {projects:[], doctrine:0, failures:0};
}

function normalizeBudgetSupportList(list, legacyCompany='', legacyAmount=0){
  const rows = Array.isArray(list) ? list.map(entry => ({
    company:entry?.company || '',
    amount:Math.max(0, Number(entry?.amount || 0)),
  })) : [];
  if(legacyCompany && legacyAmount > 0 && !rows.some(entry => entry.company === legacyCompany)){
    rows.push({company:legacyCompany, amount:Number(legacyAmount) || 0});
  }
  while(rows.length < 2) rows.push({company:'', amount:0});
  return rows.slice(0, 6);
}

function ensureStateShape(){
  if(!S.history) S.history = createEmptyHistory();
  if(!S.eventHistory) S.eventHistory = [];
  if(!S.completedGoals) S.completedGoals = createGoalState();
  if(!S.space) S.space = createSpaceState();
  if(!S.space.planets || !Object.keys(S.space.planets).length) S.space.planets = createPlanetResourceState();
  if(!S.nuclear) S.nuclear = createNuclearState();
  if(typeof S.prevMoney !== 'number') S.prevMoney = S.money;
  if(typeof S.prevPolCap !== 'number') S.prevPolCap = S.polCap;
  if(typeof S.prevPop !== 'number') S.prevPop = S.pop;
  if(typeof S.prevGdp !== 'number') S.prevGdp = S.gdp;
  if(typeof S.prevApproval !== 'number') S.prevApproval = S.approval;
  if(typeof S.prevTechLevel !== 'number') S.prevTechLevel = S.techLevel || 12;
  if(typeof S.prevYenValue !== 'number') S.prevYenValue = S.yenValue || 100;
  if(typeof S.inflation !== 'number') S.inflation = 2.1;
  if(typeof S.unemployment !== 'number') S.unemployment = 4.3;
  if(typeof S.stability !== 'number') S.stability = 68;
  if(typeof S.lastAutoSaveWeek !== 'number') S.lastAutoSaveWeek = 0;
  if(typeof S.saveVersion !== 'number') S.saveVersion = 4;
  if(typeof S.grandVictory !== 'boolean') S.grandVictory = false;
  if(typeof S.gameOverPending !== 'boolean') S.gameOverPending = false;
  if(typeof S.autoSaveEnabled !== 'boolean') S.autoSaveEnabled = true;
  if(!S.securityProfile || typeof S.securityProfile !== 'object') S.securityProfile = {};
  if(typeof S.securityProfile.protectedSaves !== 'boolean') S.securityProfile.protectedSaves = true;
  if(typeof S.securityProfile.activationRequired !== 'boolean') S.securityProfile.activationRequired = false;
  if(typeof S.securityProfile.lastIntegrityCheckWeek !== 'number') S.securityProfile.lastIntegrityCheckWeek = 0;
  if(typeof S.securityProfile.tamperWarnings !== 'number') S.securityProfile.tamperWarnings = 0;
  if(typeof S.securityProfile.optimizationProfile !== 'string') S.securityProfile.optimizationProfile = 'balanced';
  S.securityProfile.installationId = getInstallationId();
  S.securityProfile.licenseStatus = getActivationInfo().status;
  if(typeof S.nationalScore !== 'number') S.nationalScore = 0;
  if(typeof S.techLevel !== 'number') S.techLevel = 12;
  if(typeof S.technologyMomentum !== 'number') S.technologyMomentum = 0;
  if(typeof S.yenValue !== 'number') S.yenValue = 100;
  if(typeof S.businessCycle !== 'number') S.businessCycle = 0.08;
  if(typeof S.businessCyclePhase !== 'number') S.businessCyclePhase = 0;
  if(typeof S.sandboxMode !== 'boolean') S.sandboxMode = false;
  if(typeof S.difficultyMode !== 'string') S.difficultyMode = 'normal';
  if(!['easy','normal','hard'].includes(S.difficultyMode)) S.difficultyMode = 'normal';
  S.worldConfigLocked = true;
  S.polCapMax = S.sandboxMode ? 6000 : 400;
  if(!S.sandboxMode) S.polCap = Math.min(S.polCap, S.polCapMax);
  if(typeof S.language !== 'string') S.language = 'ja';
  S.language = getPreferredLanguage(S.language);
  if(typeof S.currentMapMode !== 'string') S.currentMapMode = 'japan';
  if(typeof S.mapFocus !== 'string') S.mapFocus = 'panel';
  if(typeof S.taxRate !== 'number') S.taxRate = 18;
  if(typeof S.socialUnrest !== 'number') S.socialUnrest = 18;
  if(typeof S.briberyExposure !== 'number') S.briberyExposure = 0;
  if(typeof S.ideologyScore !== 'number') S.ideologyScore = 28;
  if(typeof S.ideologyDrift !== 'number') S.ideologyDrift = 0;
  if(typeof S.lawChangeIntervalYears !== 'number') S.lawChangeIntervalYears = 3;
  if(typeof S.nextLawChangeWeek !== 'number') S.nextLawChangeWeek = 0;
  if(typeof S.nextBubbleEligibleWeek !== 'number' || Number.isNaN(S.nextBubbleEligibleWeek)) S.nextBubbleEligibleWeek = Math.max(260, (S.totalWeeks || 0) + 260);
  if(typeof S.lastLawChangeId !== 'string') S.lastLawChangeId = '';
  if(typeof S.chaosRepeatMode !== 'boolean') S.chaosRepeatMode = false;
  if(typeof S.timeMode !== 'string') S.timeMode = 'realtime';
  if(!['realtime','turn'].includes(S.timeMode)) S.timeMode = 'realtime';
  if(typeof S.realtimeSpeed !== 'number') S.realtimeSpeed = S.speed > 0 ? S.speed : 1;
  if(typeof S.turnStepWeeks !== 'number') S.turnStepWeeks = 1;
  if(![1,4,13].includes(S.turnStepWeeks)) S.turnStepWeeks = 1;
  if(typeof S.turnProcessing !== 'boolean') S.turnProcessing = false;
  if(typeof S.uiMode !== 'string') S.uiMode = 'desktop';
  if(!['desktop','mobile'].includes(S.uiMode)) S.uiMode = 'desktop';
  if(typeof S.mobileTopCollapsed !== 'boolean') S.mobileTopCollapsed = false;
  if(typeof S.mobileBottomCollapsed !== 'boolean') S.mobileBottomCollapsed = false;
  if(typeof S.mobilePanelCollapsed !== 'boolean') S.mobilePanelCollapsed = false;
  if(typeof S.mobileMapHudCollapsed !== 'boolean') S.mobileMapHudCollapsed = false;
  if(typeof S.mapTraceOpen !== 'boolean') S.mapTraceOpen = false;
  if(typeof S.mapTraceActive !== 'boolean') S.mapTraceActive = false;
  if(typeof S.mapTraceDrawing !== 'boolean') S.mapTraceDrawing = false;
  if(!Array.isArray(S.mapTracePoints)) S.mapTracePoints = [];
  if(typeof S.prefectureEnhancementBrush !== 'object') S.prefectureEnhancementBrush = null;
  if(typeof S.prefectureBrushSelection !== 'object') S.prefectureBrushSelection = null;
  if(typeof S.infraMapFilter !== 'string') S.infraMapFilter = 'all';
  if(typeof S.selectedChaosCompany !== 'string') S.selectedChaosCompany = '';
  if(typeof S.lastUsedSaveSlot !== 'string') S.lastUsedSaveSlot = 'autosave';
  if(typeof S.homeSaveExpanded !== 'boolean') S.homeSaveExpanded = false;
  if(!S.saveLabels || typeof S.saveLabels !== 'object') S.saveLabels = {};
  if(typeof S.historyMetric !== 'string') S.historyMetric = 'gdp';
  if(typeof S.historyRange !== 'number') S.historyRange = 260;
  if(!S.productIndustrialization || typeof S.productIndustrialization !== 'object') S.productIndustrialization = {};
  if(!S.productIndustrializationAnnounced || typeof S.productIndustrializationAnnounced !== 'object') S.productIndustrializationAnnounced = {};
  if(!S.productIndustrializationDetail || typeof S.productIndustrializationDetail !== 'object') S.productIndustrializationDetail = {};
  (S.unlockedProducts || []).forEach(productId => {
    if(!productCatalog?.[productId]?.requires?.length) return;
    const overall = typeof S.productIndustrialization[productId] === 'number' ? clamp(S.productIndustrialization[productId], 0, 1) : null;
    if(!S.productIndustrializationDetail[productId] || typeof S.productIndustrializationDetail[productId] !== 'object'){
      const seeded = overall ?? 1;
      S.productIndustrializationDetail[productId] = {
        certification:seeded,
        line:seeded,
        supply:seeded,
        channel:seeded,
      };
    }
    if(typeof S.productIndustrialization[productId] !== 'number'){
      const detail = S.productIndustrializationDetail[productId];
      S.productIndustrialization[productId] = clamp(((detail.certification || 0) + (detail.line || 0) + (detail.supply || 0) + (detail.channel || 0)) / 4, 0, 1);
    }
    if(S.productIndustrialization[productId] >= 1){
      S.productIndustrializationAnnounced[productId] = true;
    }
  });
  if(typeof S.historyMode !== 'string') S.historyMode = 'macro';
  if(typeof S.companyChartRange !== 'number') S.companyChartRange = 260;
  if(typeof S.companyChartMode !== 'string') S.companyChartMode = 'price';
  if(!['price','revenue','demand'].includes(S.companyChartMode)) S.companyChartMode = 'price';
  if(typeof S.majorEventCooldown !== 'number') S.majorEventCooldown = 0;
  if(!S.activeMajorEvent) S.activeMajorEvent = null;
  if(typeof S.tradeQuantityDraft !== 'number') S.tradeQuantityDraft = 5;
  if(typeof S.tradeDraftMode !== 'string') S.tradeDraftMode = 'percent';
  if(!['percent','amount'].includes(S.tradeDraftMode)) S.tradeDraftMode = 'percent';
  if(typeof S.tradeAmountDraft !== 'number' || !Number.isFinite(S.tradeAmountDraft)) S.tradeAmountDraft = 1000;
  S.tradeAmountDraft = Math.max(1, Math.round(S.tradeAmountDraft));
  if(typeof S.focusMode !== 'boolean') S.focusMode = false;
  if(typeof S.focusTrayOpen !== 'boolean') S.focusTrayOpen = false;
  if(typeof S.focusTrayPosition !== 'string') S.focusTrayPosition = 'left';
  if(!['left','bottom'].includes(S.focusTrayPosition)) S.focusTrayPosition = 'left';
  if(!Array.isArray(S.focusTrayStats) || !S.focusTrayStats.length) S.focusTrayStats = ['money','gdp','approval','yen'];
  if(typeof S.qualityMode !== 'string') S.qualityMode = 'balanced';
  if(!['low','balanced','high'].includes(S.qualityMode)) S.qualityMode = 'balanced';
  if(typeof S.newsDockCollapsed !== 'boolean') S.newsDockCollapsed = false;
  if(typeof S.historyDockCollapsed !== 'boolean') S.historyDockCollapsed = true;
  if(!S.newsDockPos || typeof S.newsDockPos !== 'object') S.newsDockPos = {left:10, top:116};
  if(!S.historyDockPos || typeof S.historyDockPos !== 'object') S.historyDockPos = {left:362, top:116};
  if(!S.mapHudOffset || typeof S.mapHudOffset !== 'object') S.mapHudOffset = {x:0,y:0};
  if(typeof S.mapHudOffset.x !== 'number') S.mapHudOffset.x = 0;
  if(typeof S.mapHudOffset.y !== 'number') S.mapHudOffset.y = 0;
  if(!S.stockCostBasis || typeof S.stockCostBasis !== 'object') S.stockCostBasis = {};
  if(!S.forcedMigrationDraft || typeof S.forcedMigrationDraft !== 'object') S.forcedMigrationDraft = {from:'tokyo', to:'osaka', amount:18};
  if(typeof S.forcedMigrationDraft.from !== 'string') S.forcedMigrationDraft.from = 'tokyo';
  if(typeof S.forcedMigrationDraft.to !== 'string') S.forcedMigrationDraft.to = 'osaka';
  if(typeof S.forcedMigrationDraft.amount !== 'number') S.forcedMigrationDraft.amount = 18;
  if(typeof S.birthRate !== 'number') S.birthRate = DEFAULT_STATE.birthRate || 7.4;
  if(typeof S.bankLoanDraftCho !== 'number') S.bankLoanDraftCho = 1000;
  if(typeof S.cyberSuccesses !== 'number') S.cyberSuccesses = 0;
  Object.entries(S.ownedStocks || {}).forEach(([name, qty]) => {
    if(typeof S.stockCostBasis[name] === 'number') return;
    const company = getCompanyByName?.(name);
    if(company) S.stockCostBasis[name] = company.price * qty;
  });
  if(typeof S.spaceFocusBody !== 'string') S.spaceFocusBody = 'moon';
  if(!S.japanView) S.japanView = {offsetX:0, offsetY:0, zoom:1};
  if(!S.worldView) S.worldView = {offsetX:0, offsetY:0, zoom:1};
  if(!S.spaceView) S.spaceView = {offsetX:-0.02, offsetY:-0.03, zoom:1.02};
  if(typeof S.worldFocusCountry !== 'string') S.worldFocusCountry = 'japan';
  if(typeof S.japanFocusPrefecture !== 'string') S.japanFocusPrefecture = 'tokyo';
  if(!S.budgetDraft) S.budgetDraft = {military:12000, education:9000, technology:11000, company:'', companySupport:0, companySupports:[{company:'', amount:0},{company:'', amount:0}]};
  if(!S.annualBudget) S.annualBudget = {military:12000, education:9000, technology:11000, company:'', companySupport:0, companySupports:[{company:'', amount:0},{company:'', amount:0}]};
  if(typeof S.budgetSetYear !== 'number') S.budgetSetYear = 2024;
  if(!S.subsidyHeat || typeof S.subsidyHeat !== 'object') S.subsidyHeat = {};
  if(!Array.isArray(S.researchProjects)) S.researchProjects = [];
  if(!Array.isArray(S.completedResearch)) S.completedResearch = [];
  if(!Array.isArray(S.unlockedProducts)) S.unlockedProducts = [];
  if(!Array.isArray(S.bioWeaponUnlocks)) S.bioWeaponUnlocks = [];
  if(!Array.isArray(S.publicWorks)) S.publicWorks = [];
  if(!Array.isArray(S.transportNetworks)) S.transportNetworks = [];
  if(!Array.isArray(S.tradeShipments)) S.tradeShipments = [];
  if(typeof S.buildRepeatMode !== 'boolean') S.buildRepeatMode = true;
  if(typeof S.transportRepeatMode !== 'boolean') S.transportRepeatMode = true;
  if(typeof S.strikeRepeatMode !== 'boolean') S.strikeRepeatMode = true;
  if(typeof S.draggingRoute !== 'object') S.draggingRoute = null;
  if(!S.foreignOwnedStocks || typeof S.foreignOwnedStocks !== 'object') S.foreignOwnedStocks = {};
  if(!Array.isArray(S.warVictories)) S.warVictories = [];
  if(typeof S.tradeFocusResource !== 'string') S.tradeFocusResource = '';
  if(typeof S.lastDividendYear !== 'number') S.lastDividendYear = Math.max(2024, S.year - 1);
  if(!S.activeWar) S.activeWar = null;
  if(!Array.isArray(S.spyMissions)) S.spyMissions = [];
  if(!S.energy) S.energy = {production:0,demand:0};
  if(!S.militaryInventory || typeof S.militaryInventory !== 'object') S.militaryInventory = {infantry:0,tanks:0,fighters:0,ships:0,drones:0,spies:0};
  if(typeof S.militaryInventory.spies !== 'number') S.militaryInventory.spies = 0;
  if(!Array.isArray(S.militaryProjects)) S.militaryProjects = [];
  if(!Array.isArray(S.mapEffects)) S.mapEffects = [];
  if(!Array.isArray(S.loans)) S.loans = [];
  if((S.totalWeeks || 0) <= 1 && S.loans.length === 0){
    S.loans.push({amount:10000000, rate:0.0009, original:10000000, termWeeks:166666, weeklyPrincipal:60, seeded:true});
  }
  if(!S.weeklyBreakdown) S.weeklyBreakdown = {tax:0,budget:0,trade:0,dividend:0,space:0,interest:0,principal:0,companySupport:0,publicRevenue:0,operations:0,socialSecurity:0,healthcare:0,childcare:0,administration:0,adjustments:0};
  if(typeof S.weeklyBreakdown.publicRevenue !== 'number') S.weeklyBreakdown.publicRevenue = 0;
  if(typeof S.weeklyBreakdown.socialSecurity !== 'number') S.weeklyBreakdown.socialSecurity = 0;
  if(typeof S.weeklyBreakdown.healthcare !== 'number') S.weeklyBreakdown.healthcare = 0;
  if(typeof S.weeklyBreakdown.childcare !== 'number') S.weeklyBreakdown.childcare = 0;
  if(typeof S.weeklyBreakdown.administration !== 'number') S.weeklyBreakdown.administration = 0;
  if(typeof S.started !== 'boolean') S.started = false;
  if(typeof S.homeMessage !== 'string') S.homeMessage = '';
  if(!Array.isArray(S.trackedPrices) || !S.trackedPrices.length) S.trackedPrices = ['copper','oil','gold','lithium','silicon'];
  if(!S.selectedChaosCompany || !companies.find(company => company.name === S.selectedChaosCompany)){
    S.selectedChaosCompany = companies[0]?.name || '';
  }
  S.publicWorks.forEach((work, index) => {
    if(!work.id) work.id = `pw-restored-${index}`;
    if(typeof work.operatingRate !== 'number') work.operatingRate = work.status === 'active' ? 75 : 100;
    if(typeof work.weeklyUsers !== 'number') work.weeklyUsers = 0;
    if(typeof work.weeklyRevenue !== 'number') work.weeklyRevenue = 0;
    if(typeof work.weeklyBalance !== 'number') work.weeklyBalance = 0;
    if(typeof work.monthlyUsers !== 'number') work.monthlyUsers = 0;
    if(typeof work.monthlyRevenue !== 'number') work.monthlyRevenue = 0;
    if(typeof work.monthlyBalance !== 'number') work.monthlyBalance = 0;
    if(typeof work.monthlyWeeks !== 'number') work.monthlyWeeks = 0;
  });
  S.transportNetworks.forEach((route, index) => {
    if(!route.id) route.id = `tn-restored-${index}`;
    if(typeof route.operatingRate !== 'number') route.operatingRate = route.status === 'active' ? 82 : 100;
    if(typeof route.weeklyUsers !== 'number') route.weeklyUsers = 0;
    if(typeof route.weeklyRevenue !== 'number') route.weeklyRevenue = 0;
    if(typeof route.weeklyBalance !== 'number') route.weeklyBalance = 0;
    if(typeof route.monthlyUsers !== 'number') route.monthlyUsers = 0;
    if(typeof route.monthlyRevenue !== 'number') route.monthlyRevenue = 0;
    if(typeof route.monthlyBalance !== 'number') route.monthlyBalance = 0;
    if(typeof route.monthlyWeeks !== 'number') route.monthlyWeeks = 0;
    if(typeof route.chainId !== 'string' || !route.chainId) route.chainId = route.id;
    if(typeof route.chainLabel !== 'string' || !route.chainLabel) route.chainLabel = '';
  });
  if(S.timeMode === 'turn'){
    S.speed = 0;
  } else if(S.speed > 0){
    S.realtimeSpeed = S.speed;
  }
  S.budgetDraft.companySupports = normalizeBudgetSupportList(S.budgetDraft.companySupports, S.budgetDraft.company, S.budgetDraft.companySupport);
  S.annualBudget.companySupports = normalizeBudgetSupportList(S.annualBudget.companySupports, S.annualBudget.company, S.annualBudget.companySupport);
  S.polCapMax = S.sandboxMode ? 6000 : 400;
  if(!S.sandboxMode) S.polCap = Math.min(S.polCap, S.polCapMax);
  Object.keys(createEmptyHistory()).forEach(k => {
    if(!Array.isArray(S.history[k])) S.history[k] = [];
  });
  ensureForeignCompanies();
}

// ============================================================
// RESOURCES - realistic quantities in tons/units
// ============================================================
const resources = {
  rice:{name:'米',amount:8500000,max:15000000,color:'#8BC34A',domestic:true,region:'tohoku',unit:'t'},
  fish:{name:'水産物',amount:3200000,max:8000000,color:'#00BCD4',domestic:true,region:'sea',unit:'t'},
  timber:{name:'木材',amount:4500000,max:10000000,color:'#795548',domestic:true,region:'hokkaido',unit:'m³'},
  copper:{name:'銅',amount:35000,max:200000,color:'#FF9800',domestic:true,region:'shikoku',price:8.50,priceHist:[],unit:'t'},
  iron:{name:'鉄鉱石',amount:500000,max:500000,color:'#607D8B',domestic:false,price:120,priceHist:[],unit:'t'},
  oil:{name:'原油',amount:50000000,max:50000000,color:'#333',domestic:false,price:75,priceHist:[],unit:'bbl'},
  gas:{name:'天然ガス',amount:20000000,max:20000000,color:'#FF5722',domestic:false,price:3.2,priceHist:[],unit:'MMBtu'},
  coal:{name:'石炭',amount:1200000,max:10000000,color:'#555',domestic:true,region:'kyushu',price:180,priceHist:[],unit:'t'},
  gold:{name:'金',amount:250,max:5000,color:'#FFD700',domestic:true,region:'tohoku',price:2050,priceHist:[],unit:'oz'},
  silver:{name:'銀',amount:1500,max:20000,color:'#C0C0C0',domestic:true,region:'chubu',price:24,priceHist:[],unit:'oz'},
  lithium:{name:'リチウム',amount:50000,max:50000,color:'#E91E63',domestic:false,price:42000,priceHist:[],unit:'t'},
  rareearth:{name:'レアアース',amount:30000,max:30000,color:'#9C27B0',domestic:false,price:310,priceHist:[],unit:'t'},
  silicon:{name:'シリコン',amount:50000,max:300000,color:'#3F51B5',domestic:true,region:'kanto',price:15,priceHist:[],unit:'t'},
  uranium:{name:'ウラン',amount:10000,max:10000,color:'#76FF03',domestic:false,price:65,priceHist:[],unit:'lb'},
  rubber:{name:'ゴム',amount:500000,max:500000,color:'#8D6E63',domestic:false,price:1.8,priceHist:[],unit:'t'},
  cotton:{name:'綿花',amount:200000,max:200000,color:'#ECEFF1',domestic:false,price:0.85,priceHist:[],unit:'t'},
  zinc:{name:'亜鉛',amount:25000,max:150000,color:'#78909C',domestic:true,region:'chubu',price:2800,priceHist:[],unit:'t'},
  tea:{name:'茶',amount:85000,max:200000,color:'#4CAF50',domestic:true,region:'chubu'},
  semiconductor:{name:'半導体素材',amount:15000,max:100000,color:'#2196F3',domestic:true,region:'kyushu',price:450,priceHist:[],unit:'wafer'},
  aluminum:{name:'アルミニウム',amount:300000,max:300000,color:'#B0BEC5',domestic:false,price:2400,priceHist:[],unit:'t'},
  titanium:{name:'チタン',amount:3000,max:50000,color:'#90CAF9',domestic:true,region:'kanto',price:11000,priceHist:[],unit:'t'},
  hydrogen:{name:'水素',amount:1000,max:100000,color:'#80DEEA',domestic:true,region:'kanto',price:8,priceHist:[],unit:'kg'},
};

// Unique discoverable materials from development
const uniqueMaterials = [
  {id:'sakurium',name:'サクリウム',color:'#FF69B4',price:85000,desc:'超伝導特性を持つ日本固有鉱物',effects:{tech:1.15,defense:1.1}},
  {id:'deepcore',name:'ディープコア結晶',color:'#00E5FF',price:120000,desc:'深海底で発見された高エネルギー結晶',effects:{energy:1.2,tech:1.1}},
  {id:'fujiite',name:'フジアイト',color:'#7C4DFF',price:95000,desc:'富士山系地下で発見された超硬質鉱物',effects:{defense:1.2,construction:1.15}},
  {id:'aether_crystal',name:'エーテル結晶',color:'#FFAB40',price:200000,desc:'量子特性を持つ未知の結晶構造',effects:{tech:1.25}},
  {id:'kaiyo_gel',name:'海洋ジェル',color:'#26C6DA',price:70000,desc:'深海生物由来の万能素材',effects:{pharma:1.2,tech:1.05}},
  {id:'neo_carbon',name:'ネオカーボン',color:'#69F0AE',price:150000,desc:'超高強度ナノカーボン新素材',effects:{auto:1.15,defense:1.15,construction:1.1}},
];

const spaceMaterials = [
  {id:'helium3',name:'ヘリウム3',color:'#fff59d',price:150000,desc:'月面で採掘できる高効率核融合燃料',effects:{energy:1.18,space:1.12,defense:1.05}},
  {id:'regolith_alloy',name:'レゴリス合金',color:'#b0bec5',price:98000,desc:'月・火星由来の耐熱建材資源',effects:{construction:1.16,space:1.1,defense:1.08}},
  {id:'iridium_x',name:'イリジウムX',color:'#ffcc80',price:180000,desc:'小惑星帯で回収される高密度超伝導鉱物',effects:{tech:1.22,space:1.18,telecom:1.1}},
  {id:'cryo_water',name:'極低温氷鉱',color:'#80deea',price:62000,desc:'火星圏で回収できる推進剤向け氷資源',effects:{space:1.16,energy:1.08}},
  {id:'exo_biomass',name:'エクソバイオマス',color:'#a5d6a7',price:124000,desc:'微生物由来の医療・農業向け宇宙資源',effects:{hospital:1.18,pharma:1.14,food:1.05}},
];

Object.values(resources).forEach(r=>{
  if(r.price){
    r.basePrice = r.price;
    r.priceHist=[];
    for(let i=0;i<1040;i++)r.priceHist.push(r.price*(0.96+Math.random()*.08));
  }
});

// ============================================================
// SECTORS & COMPANIES
// ============================================================
const sectors = {
  tech:{name:'IT・技術',color:'#2196F3',icon:'💻'},
  auto:{name:'自動車',color:'#4CAF50',icon:'🚗'},
  energy:{name:'エネルギー',color:'#FF9800',icon:'⚡'},
  finance:{name:'金融',color:'#FFD700',icon:'🏦'},
  defense:{name:'防衛',color:'#F44336',icon:'🛡'},
  pharma:{name:'医薬品',color:'#E91E63',icon:'💊'},
  construction:{name:'建設',color:'#795548',icon:'🏗'},
  shipping:{name:'海運',color:'#00BCD4',icon:'🚢'},
  food:{name:'食品',color:'#8BC34A',icon:'🍱'},
  mining:{name:'鉱業',color:'#9E9E9E',icon:'⛏'},
  telecom:{name:'通信',color:'#9C27B0',icon:'📡'},
  retail:{name:'小売',color:'#FF5722',icon:'🏪'},
  space:{name:'宇宙開発',color:'#90caf9',icon:'🚀'},
  hospital:{name:'病院・医療',color:'#66bb6a',icon:'🏥'},
};

const companies = [
  {name:'トヨダ自動車',sector:'auto',price:2800,base:2800,needs:['iron','rubber','lithium','silicon','aluminum'],region:'chubu',desc:'世界最大級の自動車メーカー'},
  {name:'ソニック・グループ',sector:'tech',price:15000,base:15000,needs:['silicon','rareearth','lithium','copper'],region:'kanto',desc:'総合エンタメ・テクノロジー企業'},
  {name:'三星重工業',sector:'defense',price:1200,base:1200,needs:['iron','copper','uranium','titanium'],region:'kanto',desc:'防衛・航空宇宙の総合メーカー'},
  {name:'東都電力HD',sector:'energy',price:800,base:800,needs:['oil','gas','uranium','coal'],region:'kanto',desc:'首都圏電力供給の中核'},
  {name:'武野薬品工業',sector:'pharma',price:4200,base:4200,needs:['rareearth','hydrogen'],region:'kinki',desc:'グローバル製薬大手'},
  {name:'日鉄製鋼',sector:'mining',price:3400,base:3400,needs:['iron','coal','zinc'],region:'kanto',desc:'日本最大の鉄鋼メーカー'},
  {name:'ニンテンドウ',sector:'tech',price:7500,base:7500,needs:['silicon','rareearth','copper'],region:'kinki',desc:'世界的ゲーム企業'},
  {name:'三星UFJ銀行',sector:'finance',price:1600,base:1600,needs:[],region:'kanto',desc:'メガバンクグループ'},
  {name:'大森組',sector:'construction',price:1800,base:1800,needs:['iron','timber','copper','aluminum'],region:'kanto',desc:'スーパーゼネコン'},
  {name:'日東郵船',sector:'shipping',price:4800,base:4800,needs:['oil'],region:'kanto',desc:'グローバル海運大手'},
  {name:'三星物産',sector:'finance',price:6200,base:6200,needs:[],region:'kanto',desc:'総合商社'},
  {name:'NTテレコム',sector:'telecom',price:170,base:170,needs:['copper','silicon'],region:'kanto',desc:'日本最大の通信企業'},
  {name:'日晴食品',sector:'food',price:3600,base:3600,needs:['rice','fish'],region:'kinki',desc:'インスタント食品大手'},
  {name:'イオーン',sector:'retail',price:3200,base:3200,needs:['cotton'],region:'kanto',desc:'総合小売グループ'},
  {name:'川崎重工機',sector:'defense',price:5500,base:5500,needs:['iron','copper','rubber','titanium'],region:'kinki',desc:'防衛・ロボット製造'},
  {name:'エネオック HD',sector:'energy',price:620,base:620,needs:['oil','gas'],region:'kanto',desc:'石油元売り最大手'},
  {name:'住倉金属鉱山',sector:'mining',price:4900,base:4900,needs:['copper','gold','silver','zinc'],region:'kinki',desc:'非鉄金属資源大手'},
  {name:'キーセンス',sector:'tech',price:62000,base:62000,needs:['silicon','rareearth'],region:'kinki',desc:'センサー・FA機器大手'},
  {name:'ファストリテール',sector:'retail',price:38000,base:38000,needs:['cotton'],region:'kanto',desc:'アパレルSPA世界大手'},
  {name:'INペックス',sector:'energy',price:2100,base:2100,needs:['oil','gas'],region:'kanto',desc:'国内最大の石油・ガス開発'},
  {name:'日立テクノ',sector:'tech',price:11000,base:11000,needs:['silicon','copper','iron','rareearth'],region:'kanto',desc:'社会インフラIT大手'},
  {name:'スバリスト',sector:'auto',price:2900,base:2900,needs:['iron','rubber','lithium','aluminum'],region:'kanto',desc:'AWD技術の自動車メーカー'},
  {name:'清永建設',sector:'construction',price:900,base:900,needs:['iron','timber'],region:'kanto',desc:'総合建設企業'},
  {name:'商船三星',sector:'shipping',price:5200,base:5200,needs:['oil'],region:'kanto',desc:'LNG輸送世界大手'},
  {name:'サイバーエージ',sector:'tech',price:1200,base:1200,needs:['silicon'],region:'kanto',desc:'ネット広告・メディア企業'},
  {name:'富通ゼネラル',sector:'tech',price:22000,base:22000,needs:['silicon','copper','rareearth'],region:'kanto',desc:'ITインフラ・サーバー大手'},
  {name:'トクヤマ化学',sector:'mining',price:3100,base:3100,needs:['silicon','zinc'],region:'chugoku',desc:'化学・シリコン素材大手'},
  {name:'住倉電工',sector:'construction',price:2400,base:2400,needs:['copper','aluminum'],region:'kinki',desc:'電線・通信インフラ'},
  {name:'オービタル重工',sector:'space',price:6800,base:6800,needs:['titanium','silicon','aluminum'],region:'kanto',desc:'ロケット・衛星機器の宇宙産業大手'},
  {name:'月宙システムズ',sector:'space',price:4200,base:4200,needs:['silicon','rareearth'],region:'kanto',desc:'月面探査・宇宙資源解析企業'},
  {name:'日本医療HD',sector:'hospital',price:2700,base:2700,needs:['hydrogen'],region:'kanto',desc:'全国病院ネットワークを持つ医療グループ'},
  {name:'桜十字メディカル',sector:'hospital',price:1800,base:1800,needs:['hydrogen','rareearth'],region:'kinki',desc:'高機能病院と遠隔医療を展開する医療法人連合'},
];

function seedCompanyModel(company){
  const sectorRevenueScale = {
    tech:0.022, telecom:0.021, defense:0.02, space:0.024, finance:0.017, hospital:0.016,
    pharma:0.019, construction:0.015, energy:0.017, auto:0.018, retail:0.014, food:0.013,
    mining:0.016, shipping:0.015
  };
  company.prevPrice = typeof company.prevPrice === 'number' ? company.prevPrice : company.price;
  company.hist = Array.isArray(company.hist) ? company.hist : [];
  company.sharesOutstanding = company.sharesOutstanding || (company.price >= 30000 ? 12000 : company.price >= 8000 ? 8000 : 5000);
  company.baseRevenue = typeof company.baseRevenue === 'number' ? company.baseRevenue : Math.round(Math.max(420, company.price * rand(5.2, 14.8)));
  company.revenue = company.revenue || company.baseRevenue;
  company.prevRevenue = typeof company.prevRevenue === 'number' ? company.prevRevenue : company.revenue;
  company.margin = company.margin || rand(0.05,0.18);
  company.volatility = company.volatility || rand(0.004, 0.016);
  company.sentiment = company.sentiment || rand(-0.01,0.01);
  company.favorability = typeof company.favorability === 'number' ? company.favorability : Math.round(rand(46, 62));
  company.techDepth = typeof company.techDepth === 'number' ? company.techDepth : Math.round(rand(18, 42) + (company.sector === 'space' ? 18 : company.sector === 'tech' ? 12 : company.sector === 'defense' ? 10 : company.sector === 'pharma' ? 8 : 0));
  company.techProgress = typeof company.techProgress === 'number' ? company.techProgress : rand(0, 18);
  company.roadmapIndex = typeof company.roadmapIndex === 'number' ? company.roadmapIndex : 0;
  company.floorBase = typeof company.floorBase === 'number' ? company.floorBase : Math.max(100, Math.round(company.base * rand(0.32, 0.58)));
  company.softCeilingMult = typeof company.softCeilingMult === 'number' ? company.softCeilingMult : rand(1.9, 2.55);
  company.overheatCooldown = typeof company.overheatCooldown === 'number' ? company.overheatCooldown : 0;
  company.floorWaveSeed = typeof company.floorWaveSeed === 'number' ? company.floorWaveSeed : rand(0, Math.PI * 2);
  company.idleBias = typeof company.idleBias === 'number' ? company.idleBias : rand(-0.28, 0.28);
  company.revenueScaleHint = typeof company.revenueScaleHint === 'number' ? company.revenueScaleHint : (sectorRevenueScale[company.sector] || 0.016) * rand(0.9, 1.12);
  company.baseDemand = typeof company.baseDemand === 'number' ? company.baseDemand : rand(92, 108);
  company.demand = typeof company.demand === 'number' ? company.demand : company.baseDemand;
  company.prevDemand = typeof company.prevDemand === 'number' ? company.prevDemand : company.demand;
  ensureRevenueWaveState(company);
  company.demandHist = Array.isArray(company.demandHist) ? company.demandHist : [];
  company.revenueHist = Array.isArray(company.revenueHist) ? company.revenueHist : [];
  if(!company.hist.length){
    company.hist.push(company.price);
  }
  if(!company.demandHist.length){
    company.demandHist.push(company.demand);
  }
  if(!company.revenueHist.length){
    company.revenueHist.push(company.revenue);
  }
}

companies.forEach(seedCompanyModel);

function createCompanyRoadmap(company){
  const templates = {
    tech:[
      {name:'次世代半導体ライン', threshold:32, boost:0.07, note:'売上と技術評価が上がる'},
      {name:'AI統合基盤', threshold:68, boost:0.09, note:'好景気局面で利益率が伸びる'},
      {name:'量子演算サービス', threshold:110, boost:0.11, note:'研究条件と輸出製品価値が向上'},
    ],
    auto:[
      {name:'電池モジュール刷新', threshold:28, boost:0.06, note:'エネルギー価格の逆風を受けにくくなる'},
      {name:'自動運転量産', threshold:64, boost:0.08, note:'景気回復時の伸びが強くなる'},
      {name:'超軽量素材採用', threshold:102, boost:0.1, note:'新素材を使うと基準株価が上がりやすい'},
    ],
    energy:[
      {name:'高効率火力制御', threshold:24, boost:0.06, note:'エネルギー危機時の収益が改善'},
      {name:'水素混焼設備', threshold:56, boost:0.08, note:'水素・ヘリウム3との連動が増す'},
      {name:'次世代送電網', threshold:94, boost:0.1, note:'国家エネルギー需給へ与える影響が増える'},
    ],
    defense:[
      {name:'高精度誘導装置', threshold:30, boost:0.06, note:'防衛力と売上が伸びる'},
      {name:'宇宙連携防衛', threshold:72, boost:0.09, note:'宇宙開発成功率に寄与する'},
      {name:'極超音速迎撃', threshold:116, boost:0.12, note:'戦争・抑止局面で基準株価が上がる'},
    ],
    space:[
      {name:'小型衛星バス', threshold:34, boost:0.08, note:'初期ロケット開発の前提になる'},
      {name:'重打上げフレーム', threshold:78, boost:0.1, note:'遠距離惑星ミッションを解禁'},
      {name:'深宇宙航法AI', threshold:124, boost:0.12, note:'宇宙成功率と採掘量が上がる'},
    ],
    hospital:[
      {name:'遠隔医療網', threshold:26, boost:0.05, note:'支持率と病院売上に寄与する'},
      {name:'宇宙医療解析', threshold:64, boost:0.08, note:'宇宙素材研究の条件が良くなる'},
      {name:'創薬自動化', threshold:108, boost:0.11, note:'研究完了後の利益率が伸びる'},
    ],
    pharma:[
      {name:'創薬データ基盤', threshold:24, boost:0.06, note:'研究開始条件が緩和される'},
      {name:'高速治験ライン', threshold:58, boost:0.08, note:'医療関連のイベントで強い'},
      {name:'遺伝子編集工場', threshold:100, boost:0.11, note:'高付加価値製品の基準収益が上がる'},
    ],
    telecom:[
      {name:'光海底ケーブル更新', threshold:26, boost:0.05, note:'サイバー防衛と輸出力に寄与する'},
      {name:'量子通信バックボーン', threshold:66, boost:0.09, note:'研究と宇宙の成功率に寄与する'},
      {name:'低遅延衛星網', threshold:106, boost:0.11, note:'宇宙・世界イベント時に売上が伸びる'},
    ],
    construction:[
      {name:'自動施工ライン', threshold:26, boost:0.05, note:'建設期間短縮と利益率改善'},
      {name:'耐災害モジュール', threshold:62, boost:0.08, note:'災害時の売上落ち込みが減る'},
      {name:'超大型拠点建設', threshold:104, boost:0.11, note:'公共事業波及が強くなる'},
    ],
    mining:[
      {name:'精錬歩留まり改善', threshold:22, boost:0.05, note:'資源不足時の落ち込みが減る'},
      {name:'深層採掘網', threshold:54, boost:0.08, note:'開発地域との連動が強くなる'},
      {name:'宇宙資源精製', threshold:96, boost:0.11, note:'宇宙素材で大きく伸びる'},
    ],
    shipping:[
      {name:'高効率船隊運用', threshold:24, boost:0.05, note:'貿易収入が改善する'},
      {name:'低燃費LNG船', threshold:58, boost:0.08, note:'エネルギー高騰に強くなる'},
      {name:'自律運航ルート', threshold:102, boost:0.1, note:'輸送時間とコストが改善する'},
    ],
    finance:[
      {name:'企業分析AI', threshold:22, boost:0.05, note:'景気回復時の利益率が伸びる'},
      {name:'為替ヘッジ網', threshold:52, boost:0.07, note:'円変動リスクに強くなる'},
      {name:'戦略投資プラットフォーム', threshold:98, boost:0.1, note:'海外投資と配当の底力が上がる'},
    ],
    retail:[
      {name:'需給予測エンジン', threshold:20, boost:0.04, note:'在庫調整が改善する'},
      {name:'自動物流拠点', threshold:54, boost:0.07, note:'輸送遅延の影響を軽減する'},
      {name:'越境販売基盤', threshold:96, boost:0.1, note:'輸出ルートとの相性が良くなる'},
    ],
    food:[
      {name:'低温流通網', threshold:20, boost:0.04, note:'災害・物流停滞に強くなる'},
      {name:'高効率加工ライン', threshold:50, boost:0.07, note:'国内資源との連動が増す'},
      {name:'機能性食品開発', threshold:90, boost:0.09, note:'景気後退でも売上が底堅い'},
    ],
  };
  const roadmap = templates[company.sector] || templates.tech;
  return roadmap.map((entry, index) => ({...entry, id:`${company.name}-${index}`}));
}

companies.forEach(company => {
  if(!Array.isArray(company.roadmap) || !company.roadmap.length) company.roadmap = createCompanyRoadmap(company);
});

function cloneTrackStages(companyName, trackId, stages){
  return stages.map((stage, index) => ({...stage, id:`${companyName}-${trackId}-${index}`}));
}

function createCompanyTrackChoices(company){
  const generic = [
    {
      id:'efficiency',
      name:'効率化',
      desc:'固定費と歩留まりを改善する安定路線',
      stages:[
        {name:'現場最適化', threshold:170, boost:0.03, note:'収益の土台が改善する'},
        {name:'工程統合', threshold:360, boost:0.05, note:'供給不足時の落ち込みがやや軽くなる'},
        {name:'自動制御網', threshold:640, boost:0.07, note:'基準株価と利益率がじわっと上がる', announce:true},
      ],
    },
    {
      id:'premium',
      name:'高付加価値',
      desc:'単価とブランドを上げる強気路線',
      stages:[
        {name:'高級ライン再編', threshold:190, boost:0.035, note:'売上単価が上がりやすくなる'},
        {name:'専用素材投入', threshold:400, boost:0.055, note:'好景気で伸びやすくなる'},
        {name:'旗艦製品刷新', threshold:720, boost:0.08, note:'強い上昇局面の天井が少し広がる', announce:true},
      ],
    },
    {
      id:'resilience',
      name:'耐久性',
      desc:'災害や不況に耐える守りの路線',
      stages:[
        {name:'代替調達網', threshold:160, boost:0.028, note:'資源ショックに少し強くなる'},
        {name:'分散拠点化', threshold:340, boost:0.05, note:'地域災害の被害を抑える'},
        {name:'危機対応OS', threshold:610, boost:0.072, note:'急落後の回復力が上がる', announce:true},
      ],
    },
  ];
  const templates = {
    defense:[
      {
        id:'infantry',
        name:'歩兵装備',
        desc:'個人装備・通信・防護を重点強化',
        stages:[
          {name:'次世代防護装具', threshold:200, boost:0.035, note:'歩兵量産の性能が上がる'},
          {name:'小隊通信網', threshold:430, boost:0.055, note:'歩兵系の防衛効率が上がる'},
          {name:'統合戦術スーツ', threshold:760, boost:0.085, note:'歩兵系の軍事生産に大きく効く', announce:true},
        ],
      },
      {
        id:'armor',
        name:'機甲戦力',
        desc:'戦車・装甲車・重装備を強化',
        stages:[
          {name:'複合装甲', threshold:220, boost:0.04, note:'戦車生産の性能が上がる'},
          {name:'火器管制AI', threshold:470, boost:0.06, note:'戦車と自走火器の効率が改善'},
          {name:'主力戦車刷新', threshold:820, boost:0.09, note:'機甲ユニットの完成度が大幅上昇', announce:true},
        ],
      },
      {
        id:'air',
        name:'航空優勢',
        desc:'戦闘機・迎撃・誘導装置を強化',
        stages:[
          {name:'高精度誘導装置', threshold:230, boost:0.04, note:'航空機の性能が上がる'},
          {name:'統合レーダー', threshold:500, boost:0.065, note:'制空系ユニットの完成度が上昇'},
          {name:'次世代戦闘機体系', threshold:860, boost:0.095, note:'航空ユニットの軍事生産に大きく効く', announce:true},
        ],
      },
    ],
    tech:[
      {
        id:'cyber',
        name:'サイバー',
        desc:'侵入・防衛・解析を高める情報戦路線',
        stages:[
          {name:'侵入解析基盤', threshold:180, boost:0.03, note:'サイバー投資効率が少し上がる'},
          {name:'自動防衛網', threshold:390, boost:0.05, note:'相手国より劣勢でも持ちこたえやすい'},
          {name:'国家級監視AI', threshold:700, boost:0.08, note:'サイバーと貿易条件に大きく効く', announce:true},
        ],
      },
      {
        id:'drone',
        name:'無人機',
        desc:'ドローン・自律兵器・制御系を強化',
        stages:[
          {name:'自律飛行制御', threshold:190, boost:0.034, note:'ドローン量産の質が上がる'},
          {name:'群制御ソフト', threshold:410, boost:0.055, note:'軍事・物流の連携が良くなる'},
          {name:'統合無人戦闘群', threshold:740, boost:0.085, note:'無人機系の軍事生産に大きく効く', announce:true},
        ],
      },
      {
        id:'ai',
        name:'産業AI',
        desc:'生産最適化と高収益化を狙う路線',
        stages:[
          {name:'需要予測AI', threshold:175, boost:0.032, note:'売上のブレが少し穏やかになる'},
          {name:'全社統合AI', threshold:385, boost:0.052, note:'基準売上の伸びが少し強くなる'},
          {name:'自律経営支援', threshold:690, boost:0.082, note:'長期の基準値上昇に効く', announce:true},
        ],
      },
    ],
    space:[
      {
        id:'launch',
        name:'打上げ',
        desc:'ロケット機体とエンジンを強化',
        stages:[
          {name:'軽量推進段', threshold:220, boost:0.038, note:'ロケット開発に寄与する'},
          {name:'重打上げ骨格', threshold:470, boost:0.06, note:'遠征条件を少し緩和する'},
          {name:'再使用打上げ機', threshold:860, boost:0.095, note:'宇宙遠征の成功率に大きく効く', announce:true},
        ],
      },
      {
        id:'navigation',
        name:'航法',
        desc:'誘導・航法・深宇宙制御を強化',
        stages:[
          {name:'精密航法計算', threshold:210, boost:0.036, note:'宇宙遠征の失敗率をやや抑える'},
          {name:'自律航法AI', threshold:450, boost:0.058, note:'長距離遠征に強くなる'},
          {name:'深宇宙誘導網', threshold:820, boost:0.09, note:'難関天体での成功率に大きく効く', announce:true},
        ],
      },
      {
        id:'orbital',
        name:'軌道生産',
        desc:'宇宙素材の回収と加工を伸ばす路線',
        stages:[
          {name:'軌道加工装置', threshold:205, boost:0.034, note:'宇宙資源回収量が上がる'},
          {name:'軌道工場モジュール', threshold:445, boost:0.057, note:'宇宙部門の収益が増える'},
          {name:'深宇宙製造網', threshold:810, boost:0.092, note:'宇宙産業の基準値を押し上げる', announce:true},
        ],
      },
    ],
    auto:[
      {
        id:'mobility',
        name:'次世代車両',
        desc:'民生車・高効率モビリティを強化',
        stages:[
          {name:'次世代車台', threshold:175, boost:0.03, note:'自動車の基礎収益が改善'},
          {name:'自動運転量産', threshold:385, boost:0.05, note:'景気回復局面で伸びやすい'},
          {name:'主力電動車刷新', threshold:700, boost:0.078, note:'基準株価の上昇力が増す', announce:true},
        ],
      },
      {
        id:'armor',
        name:'装甲車両',
        desc:'戦車・装甲車への転用技術を強化',
        stages:[
          {name:'重耐久車台', threshold:210, boost:0.036, note:'戦車系の軍事生産に寄与する'},
          {name:'軍民共通ライン', threshold:440, boost:0.058, note:'戦車量産の週数を短縮する'},
          {name:'重機甲統合設計', threshold:790, boost:0.088, note:'機甲戦力の完成度に大きく効く', announce:true},
        ],
      },
      {
        id:'logistics',
        name:'補給車両',
        desc:'トラック・輸送・補給最適化に特化',
        stages:[
          {name:'高耐久輸送車', threshold:165, boost:0.028, note:'補給線の強化に寄与する'},
          {name:'自動物流群', threshold:360, boost:0.05, note:'輸送演出と実効効率が少し改善'},
          {name:'全地形補給網', threshold:650, boost:0.078, note:'戦争と物流の底力が上がる', announce:true},
        ],
      },
    ],
    construction:[
      {
        id:'civil',
        name:'大型土木',
        desc:'通常の公共工事と大型構築を強化',
        stages:[
          {name:'大型施工機', threshold:170, boost:0.03, note:'建設完了後の波及が少し強くなる'},
          {name:'自動施工網', threshold:370, boost:0.05, note:'工期短縮に寄与する'},
          {name:'超大型造成技術', threshold:670, boost:0.08, note:'公共事業の経済押上げが大きくなる', announce:true},
        ],
      },
      {
        id:'fortify',
        name:'防災要塞',
        desc:'耐災害・軍用施設・堅牢化を強化',
        stages:[
          {name:'耐震構造刷新', threshold:180, boost:0.032, note:'災害時の被害を抑える'},
          {name:'要塞化基準', threshold:390, boost:0.052, note:'防衛施設と基地建設に寄与'},
          {name:'国家級防災都市', threshold:710, boost:0.082, note:'長期災害の回復力がかなり上がる', announce:true},
        ],
      },
      {
        id:'logistics',
        name:'物流基盤',
        desc:'道路・港・輸送拠点を強化',
        stages:[
          {name:'高速積替網', threshold:170, boost:0.03, note:'貿易と内需の連携が良くなる'},
          {name:'港湾自動化', threshold:380, boost:0.05, note:'輸送の停滞を少し減らす'},
          {name:'全国物流OS', threshold:700, boost:0.08, note:'国内物流と補給線に大きく効く', announce:true},
        ],
      },
    ],
    shipping:[
      {
        id:'naval',
        name:'艦艇',
        desc:'海上戦力と大型艦の整備を強化',
        stages:[
          {name:'護衛艦改修', threshold:205, boost:0.036, note:'艦艇の生産効率が上がる'},
          {name:'大型艦体設計', threshold:450, boost:0.058, note:'艦艇の性能と週数が改善'},
          {name:'次世代海上打撃群', threshold:810, boost:0.09, note:'艦艇系の軍事生産に大きく効く', announce:true},
        ],
      },
      {
        id:'freight',
        name:'物流船団',
        desc:'交易路と海上輸送の安定化を狙う',
        stages:[
          {name:'高効率船隊運用', threshold:165, boost:0.028, note:'輸送コストが少し軽くなる'},
          {name:'自動積載管理', threshold:350, boost:0.05, note:'貿易便の回転が良くなる'},
          {name:'超大規模港湾連携', threshold:650, boost:0.078, note:'長距離輸送の底力が上がる', announce:true},
        ],
      },
      {
        id:'autonomous',
        name:'自律運航',
        desc:'無人輸送と危険海域対応を強化',
        stages:[
          {name:'自律航行補助', threshold:175, boost:0.03, note:'輸送遅延の影響がやや減る'},
          {name:'群航行制御', threshold:380, boost:0.052, note:'戦時物流の安定化に寄与'},
          {name:'全海域自動運航', threshold:690, boost:0.082, note:'物流と艦隊の両面に効く', announce:true},
        ],
      },
    ],
  };
  const selected = templates[company.sector] || generic;
  return selected.map(track => ({
    ...track,
    stages:cloneTrackStages(company.name, track.id, track.stages),
  }));
}

function getRoadmapStateKey(trackId=''){
  return trackId || '__default';
}

function ensureCompanyRoadmapState(company){
  if(!company || typeof company !== 'object') return;
  if(!company.trackProgressState || typeof company.trackProgressState !== 'object') company.trackProgressState = {};
  const loadedKey = company.loadedRoadmapTrackKey || getRoadmapStateKey(company.selectedTechTrack);
  if(!company.trackProgressState[loadedKey]){
    company.trackProgressState[loadedKey] = {
      index:typeof company.roadmapIndex === 'number' ? company.roadmapIndex : 0,
      progress:typeof company.techProgress === 'number' ? company.techProgress : 0,
    };
  }
  const defaultKey = getRoadmapStateKey('');
  if(!company.trackProgressState[defaultKey]){
    company.trackProgressState[defaultKey] = {
      index:typeof company.roadmapIndex === 'number' ? company.roadmapIndex : 0,
      progress:typeof company.techProgress === 'number' ? company.techProgress : 0,
    };
  }
  (company.techTracks || []).forEach(track => {
    const key = getRoadmapStateKey(track.id);
    if(!company.trackProgressState[key]){
      company.trackProgressState[key] = {index:0, progress:0};
    }
  });
}

function persistCompanyRoadmapState(company, key=company?.loadedRoadmapTrackKey || getRoadmapStateKey(company?.selectedTechTrack || '')){
  if(!company) return;
  ensureCompanyRoadmapState(company);
  company.trackProgressState[key] = {
    index:Math.max(0, Math.floor(company.roadmapIndex || 0)),
    progress:Math.max(0, Number(company.techProgress || 0)),
  };
}

function syncCompanyRoadmap(company){
  if(!Array.isArray(company.techTracks) || !company.techTracks.length){
    company.techTracks = createCompanyTrackChoices(company);
  }
  if(typeof company.selectedTechTrack !== 'string'){
    company.selectedTechTrack = '';
  }
  if(company.selectedTechTrack && !company.techTracks.some(track => track.id === company.selectedTechTrack)){
    company.selectedTechTrack = '';
  }
  ensureCompanyRoadmapState(company);
  const nextKey = getRoadmapStateKey(company.selectedTechTrack);
  const loadedKey = company.loadedRoadmapTrackKey || nextKey;
  if(loadedKey !== nextKey){
    persistCompanyRoadmapState(company, loadedKey);
    const nextState = company.trackProgressState[nextKey] || {index:0, progress:0};
    company.roadmapIndex = nextState.index || 0;
    company.techProgress = nextState.progress || 0;
  } else {
    const currentState = company.trackProgressState[nextKey] || {index:0, progress:0};
    if(typeof company.roadmapIndex !== 'number') company.roadmapIndex = currentState.index || 0;
    if(typeof company.techProgress !== 'number') company.techProgress = currentState.progress || 0;
  }
  const activeTrack = company.techTracks.find(track => track.id === company.selectedTechTrack) || null;
  company.roadmap = activeTrack ? activeTrack.stages.map(stage => ({...stage})) : createCompanyRoadmap(company);
  company.roadmapIndex = clamp(company.roadmapIndex || 0, 0, company.roadmap.length);
  company.loadedRoadmapTrackKey = nextKey;
  persistCompanyRoadmapState(company, nextKey);
}

function ensureCompanySystems(){
  const sectorRevenueScale = {
    tech:0.022, telecom:0.021, defense:0.02, space:0.024, finance:0.017, hospital:0.016,
    pharma:0.019, construction:0.015, energy:0.017, auto:0.018, retail:0.014, food:0.013,
    mining:0.016, shipping:0.015
  };
  companies.forEach(company => {
    if(typeof company.floorBase !== 'number') company.floorBase = Math.max(100, Math.round(company.base * rand(0.32, 0.58)));
    if(typeof company.softCeilingMult !== 'number') company.softCeilingMult = rand(1.9, 2.55);
    if(typeof company.overheatCooldown !== 'number') company.overheatCooldown = 0;
    if(typeof company.floorWaveSeed !== 'number') company.floorWaveSeed = rand(0, Math.PI * 2);
    if(typeof company.idleBias !== 'number') company.idleBias = rand(-0.28, 0.28);
    if(typeof company.revenueScaleHint !== 'number') company.revenueScaleHint = (sectorRevenueScale[company.sector] || 0.016) * rand(0.9, 1.12);
    if(typeof company.baseDemand !== 'number') company.baseDemand = rand(92, 108);
    if(typeof company.demand !== 'number') company.demand = company.baseDemand;
    if(typeof company.prevDemand !== 'number') company.prevDemand = company.demand;
    ensureRevenueWaveState(company);
    if(!Array.isArray(company.demandHist)) company.demandHist = [company.demand];
    if(typeof company.techProgress !== 'number') company.techProgress = rand(0, 12);
    if(typeof company.roadmapIndex !== 'number') company.roadmapIndex = 0;
    if(typeof company.techDepth !== 'number') company.techDepth = 20;
    if(typeof company.favorability !== 'number') company.favorability = 50;
    if(typeof company.baseRevenue !== 'number') company.baseRevenue = Math.max(420, Math.round((company.base || company.price || 100) * rand(4.8, 13.2)));
    if(!Number.isFinite(company.revenue) || company.revenue <= 0) company.revenue = company.baseRevenue;
    if(company.revenue > company.baseRevenue * 18 && (S.totalWeeks || 0) < 8){
      company.revenue = Math.round(company.baseRevenue * rand(0.96, 1.08));
    }
    if(typeof company.recoveryLag !== 'number') company.recoveryLag = 0;
    if(typeof company.patternMode !== 'string') company.patternMode = pick(['range','triangle-up','triangle-down','flag-up','flag-down','bottoming','topping']);
    if(typeof company.patternAge !== 'number') company.patternAge = Math.floor(rand(0, 6));
    if(typeof company.patternLength !== 'number') company.patternLength = Math.floor(rand(6, 15));
    if(typeof company.patternCompression !== 'number') company.patternCompression = rand(0.52, 1.12);
    if(typeof company.patternBias !== 'number') company.patternBias = rand(-0.4, 0.4);
    if(typeof company.patternPhaseSeed !== 'number') company.patternPhaseSeed = rand(0, Math.PI * 2);
    if(typeof company.mapLayoutVersion !== 'number') company.mapLayoutVersion = 0;
    if(typeof company.loadedRoadmapTrackKey !== 'string') company.loadedRoadmapTrackKey = getRoadmapStateKey(company.selectedTechTrack || '');
    syncCompanyRoadmap(company);
  });
}

ensureCompanySystems();

// ============================================================
// REGIONS
// ============================================================
const regions = {
  hokkaido:{name:'北海道',cx:.72,cy:.14,resources:['timber','coal','fish'],devSlots:30,developed:0},
  tohoku:{name:'東北',cx:.71,cy:.30,resources:['rice','gold'],devSlots:30,developed:0},
  kanto:{name:'関東',cx:.68,cy:.44,resources:['silicon','semiconductor','titanium','hydrogen'],devSlots:30,developed:0},
  chubu:{name:'中部',cx:.58,cy:.47,resources:['silver','zinc','tea'],devSlots:30,developed:0},
  kinki:{name:'近畿',cx:.50,cy:.52,resources:['copper'],devSlots:30,developed:0},
  chugoku:{name:'中国地方',cx:.38,cy:.54,resources:['iron'],devSlots:30,developed:0},
  shikoku:{name:'四国',cx:.42,cy:.62,resources:['copper'],devSlots:30,developed:0},
  kyushu:{name:'九州',cx:.28,cy:.64,resources:['coal','semiconductor'],devSlots:30,developed:0},
  okinawa:{name:'沖縄',cx:.15,cy:.85,resources:['fish'],devSlots:30,developed:0},
  sea_pacific:{name:'太平洋沖',cx:.82,cy:.65,resources:['oil','gas','fish'],devSlots:30,developed:0,isSea:true},
  sea_japan:{name:'日本海沖',cx:.40,cy:.22,resources:['gas','fish'],devSlots:30,developed:0,isSea:true},
  sea_east:{name:'東シナ海',cx:.18,cy:.72,resources:['oil','rareearth'],devSlots:30,developed:0,isSea:true},
};
Object.values(regions).forEach(region => {
  if(!region.disaster) region.disaster = null;
});

const prefectures = [
  ['hokkaido','北海道','hokkaido',0.735,0.148],
  ['aomori','青森','tohoku',0.72,0.23],
  ['iwate','岩手','tohoku',0.73,0.29],
  ['miyagi','宮城','tohoku',0.72,0.35],
  ['akita','秋田','tohoku',0.68,0.29],
  ['yamagata','山形','tohoku',0.68,0.35],
  ['fukushima','福島','tohoku',0.69,0.41],
  ['ibaraki','茨城','kanto',0.72,0.44],
  ['tochigi','栃木','kanto',0.69,0.43],
  ['gunma','群馬','kanto',0.66,0.42],
  ['saitama','埼玉','kanto',0.67,0.45],
  ['chiba','千葉','kanto',0.73,0.46],
  ['tokyo','東京','kanto',0.67,0.47],
  ['kanagawa','神奈川','kanto',0.66,0.49],
  ['niigata','新潟','chubu',0.62,0.36],
  ['toyama','富山','chubu',0.58,0.41],
  ['ishikawa','石川','chubu',0.55,0.41],
  ['fukui','福井','chubu',0.53,0.44],
  ['yamanashi','山梨','chubu',0.62,0.46],
  ['nagano','長野','chubu',0.58,0.45],
  ['gifu','岐阜','chubu',0.56,0.49],
  ['shizuoka','静岡','chubu',0.63,0.50],
  ['aichi','愛知','chubu',0.58,0.52],
  ['mie','三重','kinki',0.53,0.53],
  ['shiga','滋賀','kinki',0.51,0.51],
  ['kyoto','京都','kinki',0.49,0.50],
  ['osaka','大阪','kinki',0.50,0.53],
  ['hyogo','兵庫','kinki',0.46,0.52],
  ['nara','奈良','kinki',0.52,0.54],
  ['wakayama','和歌山','kinki',0.50,0.57],
  ['tottori','鳥取','chugoku',0.41,0.49],
  ['shimane','島根','chugoku',0.37,0.49],
  ['okayama','岡山','chugoku',0.42,0.52],
  ['hiroshima','広島','chugoku',0.37,0.53],
  ['yamaguchi','山口','chugoku',0.33,0.53],
  ['tokushima','徳島','shikoku',0.44,0.60],
  ['kagawa','香川','shikoku',0.42,0.58],
  ['ehime','愛媛','shikoku',0.38,0.60],
  ['kochi','高知','shikoku',0.40,0.64],
  ['fukuoka','福岡','kyushu',0.29,0.60],
  ['saga','佐賀','kyushu',0.26,0.61],
  ['nagasaki','長崎','kyushu',0.23,0.60],
  ['kumamoto','熊本','kyushu',0.28,0.64],
  ['oita','大分','kyushu',0.31,0.62],
  ['miyazaki','宮崎','kyushu',0.30,0.67],
  ['kagoshima','鹿児島','kyushu',0.26,0.69],
  ['okinawa_pref','沖縄','okinawa',0.15,0.85],
].map(([id, name, region, x, y]) => ({id, name, region, x, y}));
const prefectureMap = Object.fromEntries(prefectures.map(prefecture => [prefecture.id, prefecture]));
const PREFECTURE_MAJOR_CITIES = {
  hokkaido:{name:'札幌', dx:-0.003, dy:-0.001},
  aomori:{name:'青森'}, iwate:{name:'盛岡'}, miyagi:{name:'仙台'}, akita:{name:'秋田'}, yamagata:{name:'山形'}, fukushima:{name:'福島'},
  ibaraki:{name:'水戸'}, tochigi:{name:'宇都宮'}, gunma:{name:'前橋'}, saitama:{name:'さいたま', dx:-0.0006}, chiba:{name:'千葉', dx:-0.0008}, tokyo:{name:'東京', dx:-0.0014}, kanagawa:{name:'横浜', dx:-0.0012},
  niigata:{name:'新潟'}, toyama:{name:'富山'}, ishikawa:{name:'金沢'}, fukui:{name:'福井'}, yamanashi:{name:'甲府'}, nagano:{name:'長野'}, gifu:{name:'岐阜'}, shizuoka:{name:'静岡'}, aichi:{name:'名古屋'},
  mie:{name:'津'}, shiga:{name:'大津'}, kyoto:{name:'京都'}, osaka:{name:'大阪', dx:-0.0007}, hyogo:{name:'神戸', dx:-0.0008}, nara:{name:'奈良'}, wakayama:{name:'和歌山'},
  tottori:{name:'鳥取'}, shimane:{name:'松江'}, okayama:{name:'岡山'}, hiroshima:{name:'広島'}, yamaguchi:{name:'山口'},
  tokushima:{name:'徳島'}, kagawa:{name:'高松'}, ehime:{name:'松山'}, kochi:{name:'高知'},
  fukuoka:{name:'福岡', dx:-0.0006}, saga:{name:'佐賀'}, nagasaki:{name:'長崎', dx:-0.0007}, kumamoto:{name:'熊本'}, oita:{name:'大分'}, miyazaki:{name:'宮崎'}, kagoshima:{name:'鹿児島'},
  okinawa_pref:{name:'那覇', dx:0.0003, dy:0.0005},
};

function getPrefectureTransitAnchor(prefectureId){
  const prefecture = prefectureMap[prefectureId];
  if(!prefecture) return null;
  const cfg = PREFECTURE_MAJOR_CITIES[prefectureId] || {};
  const fallbackX = prefecture.x;
  const fallbackY = prefecture.y;
  const projected = projectPointIntoRegionLand(
    prefecture.x + (cfg.dx || 0),
    prefecture.y + (cfg.dy || 0),
    prefecture.region,
    fallbackX,
    fallbackY
  );
  return {
    id:prefecture.id,
    prefectureId:prefecture.id,
    prefectureName:prefecture.name,
    region:prefecture.region,
    name:cfg.name || `${prefecture.name}市`,
    x:projected.x,
    y:projected.y,
  };
}

function getClosestTransportAnchor(x, y, threshold=0.05){
  const closest = prefectures
    .map(prefecture => {
      const anchor = getPrefectureTransitAnchor(prefecture.id);
      return anchor ? {anchor, dist:Math.hypot(x - anchor.x, y - anchor.y)} : null;
    })
    .filter(Boolean)
    .sort((a, b) => a.dist - b.dist)[0];
  return closest && closest.dist <= threshold ? closest.anchor : null;
}

function syncRegionCentersToPrefectures(){
  const grouped = prefectures.reduce((acc, prefecture) => {
    if(!acc[prefecture.region]) acc[prefecture.region] = [];
    acc[prefecture.region].push(prefecture);
    return acc;
  }, {});
  Object.entries(grouped).forEach(([regionId, list]) => {
    if(!regions[regionId] || !list.length) return;
    regions[regionId].cx = list.reduce((sum, prefecture) => sum + prefecture.x, 0) / list.length;
    regions[regionId].cy = list.reduce((sum, prefecture) => sum + prefecture.y, 0) / list.length;
  });
}

function getRegionLandPolygon(regionId){
  if(!regionId) return null;
  return japanPolys[regionId] || null;
}

function projectPointIntoRegionLand(x, y, regionId, fallbackX, fallbackY){
  const polygon = getRegionLandPolygon(regionId);
  const fallback = {x:fallbackX, y:fallbackY};
  if(!polygon) return fallback;
  const original = {x, y};
  if(pointInPolygon(original, polygon)) return original;
  for(let t = 0.08; t <= 1; t += 0.08){
    const candidate = {
      x:lerp(x, fallback.x, t),
      y:lerp(y, fallback.y, t),
    };
    if(pointInPolygon(candidate, polygon)) return candidate;
  }
  for(let radius = 0.002; radius <= 0.03; radius += 0.0025){
    for(let step = 0; step < 18; step++){
      const angle = (Math.PI * 2 * step) / 18;
      const candidate = {
        x:fallback.x + Math.cos(angle) * radius,
        y:fallback.y + Math.sin(angle) * radius,
      };
      if(pointInPolygon(candidate, polygon)) return candidate;
    }
  }
  return fallback;
}

syncRegionCentersToPrefectures();

const companyPrefectureHints = {
  'トヨダ自動車':'aichi',
  'ソニック・グループ':'tokyo',
  '三星重工業':'kanagawa',
  '東都電力HD':'tokyo',
  '武野薬品工業':'osaka',
  '日鉄製鋼':'ibaraki',
  'ニンテンドウ':'kyoto',
  '三星UFJ銀行':'tokyo',
  '大森組':'saitama',
  '日東郵船':'kanagawa',
  '三星物産':'tokyo',
  'NTテレコム':'tokyo',
  '日晴食品':'osaka',
  'イオーン':'chiba',
  '川崎重工機':'hyogo',
  'エネオック HD':'chiba',
  '住倉金属鉱山':'ehime',
  'キーセンス':'osaka',
  'ファストリテール':'yamaguchi',
  'INペックス':'niigata',
  '日立テクノ':'ibaraki',
  'スバリスト':'gunma',
  '清永建設':'tokyo',
  '商船三星':'tokyo',
  'サイバーエージ':'tokyo',
  '富通ゼネラル':'kanagawa',
  'トクヤマ化学':'yamaguchi',
  '住倉電工':'osaka',
  'オービタル重工':'ibaraki',
  '月宙システムズ':'fukushima',
  '日本医療HD':'tokyo',
  '桜十字メディカル':'osaka',
};

function assignCompanyPrefectures(){
  const prefecturesByRegion = prefectures.reduce((acc, prefecture) => {
    if(!acc[prefecture.region]) acc[prefecture.region] = [];
    acc[prefecture.region].push(prefecture);
    return acc;
  }, {});
  const regionUsage = {};
  companies.forEach(company => {
    const hinted = prefectureMap[company.prefecture] || prefectureMap[companyPrefectureHints[company.name]];
    if(hinted && hinted.region === company.region){
      company.prefecture = hinted.id;
      return;
    }
    const list = prefecturesByRegion[company.region] || prefectures;
    const useIndex = regionUsage[company.region] || 0;
    company.prefecture = list[useIndex % list.length].id;
    regionUsage[company.region] = useIndex + 1;
  });
}

function getSyntheticCompanySector(prefecture){
  const sectorPools = {
    hokkaido:['food','energy','telecom','space'],
    tohoku:['food','construction','energy','telecom','tech'],
    kanto:['finance','tech','telecom','retail','space'],
    chubu:['auto','construction','tech','mining','telecom'],
    kinki:['tech','finance','pharma','telecom','retail'],
    chugoku:['construction','mining','shipping','tech'],
    shikoku:['food','telecom','hospital','shipping'],
    kyushu:['energy','shipping','tech','construction','auto'],
    okinawa:['shipping','retail','hospital','telecom'],
  };
  const pool = sectorPools[prefecture.region] || ['tech','construction','retail'];
  const seed = prefecture.name.charCodeAt(0) + prefecture.name.charCodeAt(prefecture.name.length - 1);
  return pool[seed % pool.length];
}

function buildSyntheticPrefectureCompany(prefecture){
  const sector = getSyntheticCompanySector(prefecture);
  const suffixMap = {
    tech:'デジタル',
    auto:'モーターズ',
    energy:'エナジー',
    finance:'バンク',
    defense:'シールド',
    pharma:'バイオ',
    construction:'ビルド',
    shipping:'マリン',
    food:'フーズ',
    mining:'メタル',
    telecom:'ネット',
    retail:'ストア',
    space:'オービット',
    hospital:'メディカル',
  };
  const needsMap = {
    tech:['silicon','copper'],
    auto:['iron','rubber','aluminum'],
    energy:['oil','gas'],
    finance:[],
    defense:['iron','copper','titanium'],
    pharma:['hydrogen','rareearth'],
    construction:['iron','timber','copper'],
    shipping:['oil'],
    food:['rice','fish'],
    mining:['iron','zinc'],
    telecom:['copper','silicon'],
    retail:['cotton'],
    space:['titanium','silicon','rareearth'],
    hospital:['hydrogen'],
  };
  const basePrice = Math.round(rand(420, 4200));
  return {
    name:`${prefecture.name}${suffixMap[sector] || 'ワークス'}`,
    sector,
    price:basePrice,
    base:basePrice,
    needs:[...(needsMap[sector] || [])],
    region:prefecture.region,
    prefecture:prefecture.id,
    syntheticPrefectureCompany:true,
    desc:`${prefecture.name} を拠点にする ${sectors[sector]?.name || sector} 企業`,
  };
}

function ensurePrefectureCompanyCoverage(){
  assignCompanyPrefectures();
  const covered = new Set(companies.map(company => company.prefecture).filter(Boolean));
  const additions = [];
  prefectures.forEach(prefecture => {
    if(covered.has(prefecture.id)) return;
    const company = buildSyntheticPrefectureCompany(prefecture);
    seedCompanyModel(company);
    additions.push(company);
    covered.add(prefecture.id);
  });
  if(additions.length){
    additions.forEach(company => companies.push(company));
    ensureCompanySystems();
  }
  assignCompanyPrefectures();
}

let runtimeCacheEpoch = 0;
let prefectureCompanyCacheEpoch = -1;
let prefectureCompanyCacheKey = '';
let prefectureCompanyCache = new Map();
let prefectureRouteCacheEpoch = -1;
let prefectureRouteCacheKey = '';
let prefectureRouteCache = new Map();
let transportRouteGroupCacheEpoch = -1;
let transportRouteGroupCacheKey = '';
let transportRouteGroupCache = [];

function invalidateRuntimeCaches(reason=''){
  runtimeCacheEpoch++;
  prefectureCompanyCacheEpoch = -1;
  prefectureCompanyCacheKey = '';
  prefectureRouteCacheEpoch = -1;
  prefectureRouteCacheKey = '';
  transportRouteGroupCacheEpoch = -1;
  transportRouteGroupCacheKey = '';
  prefectureCompanyCache = new Map();
  prefectureRouteCache = new Map();
  transportRouteGroupCache = [];
  return reason;
}

function ensurePrefectureCompanyCache(){
  const signature = companies.map(company => `${company.name}:${company.prefecture || ''}`).join('|');
  if(prefectureCompanyCacheEpoch === runtimeCacheEpoch && prefectureCompanyCacheKey === signature) return;
  prefectureCompanyCacheEpoch = runtimeCacheEpoch;
  prefectureCompanyCacheKey = signature;
  prefectureCompanyCache = new Map();
  companies.forEach(company => {
    const key = company.prefecture || '';
    if(!key) return;
    if(!prefectureCompanyCache.has(key)) prefectureCompanyCache.set(key, []);
    prefectureCompanyCache.get(key).push(company);
  });
}

function getPrefectureCompanies(prefectureId){
  ensurePrefectureCompanyCache();
  return prefectureCompanyCache.get(prefectureId) || [];
}

function getClosestPrefecture(x, y, threshold=0.032){
  const closest = prefectures
    .map(prefecture => ({prefecture, dist:Math.hypot(x - prefecture.x, y - prefecture.y)}))
    .sort((a,b) => a.dist - b.dist)[0];
  return closest && closest.dist <= threshold ? closest.prefecture : null;
}

function resolveTransportEndpointPrefecture(mx, my, worldPos){
  const markerHit = getPrefectureMarkerAtCanvas(mx, my);
  if(markerHit) return markerHit;
  const anchorHit = getClosestTransportAnchor(worldPos.x, worldPos.y, S.mapZoom >= 1.3 ? 0.082 : 0.108);
  if(anchorHit && prefectureMap[anchorHit.prefectureId]) return prefectureMap[anchorHit.prefectureId];
  const strictPrefecture = getClosestPrefecture(worldPos.x, worldPos.y, S.mapZoom >= 1.3 ? 0.044 : 0.064);
  if(strictPrefecture) return strictPrefecture;
  return getClosestPrefecture(worldPos.x, worldPos.y, 0.092);
}

function getTransportRouteEndpoints(route){
  return {
    from:getPrefectureTransitAnchor(route.fromPrefectureId) || null,
    to:getPrefectureTransitAnchor(route.toPrefectureId) || null,
  };
}

function ensurePrefectureTransportCache(){
  const signature = (S.transportNetworks || []).map(route => `${route.id}:${route.fromPrefectureId || ''}:${route.toPrefectureId || ''}:${route.status}:${route.chainId || ''}`).join('|');
  if(prefectureRouteCacheEpoch === runtimeCacheEpoch && prefectureRouteCacheKey === signature) return;
  prefectureRouteCacheEpoch = runtimeCacheEpoch;
  prefectureRouteCacheKey = signature;
  prefectureRouteCache = new Map();
  if(!Array.isArray(S.transportNetworks)) return;
  S.transportNetworks.forEach(route => {
    [route.fromPrefectureId, route.toPrefectureId].forEach(prefectureId => {
      if(!prefectureId) return;
      if(!prefectureRouteCache.has(prefectureId)) prefectureRouteCache.set(prefectureId, []);
      prefectureRouteCache.get(prefectureId).push(route);
    });
  });
}

function getPrefectureTransportRoutes(prefectureId, activeOnly=false){
  ensurePrefectureTransportCache();
  const routes = prefectureRouteCache.get(prefectureId) || [];
  return activeOnly ? routes.filter(route => route.status === 'active') : routes;
}

function getTransportRouteChainId(route){
  return route?.chainId || route?.id || '';
}

function getTransportRouteGroups(){
  const signature = (S.transportNetworks || []).map(route => `${route.id}:${route.chainId || ''}:${route.status}:${route.weeksLeft || 0}:${route.operatingRate || 0}`).join('|');
  if(transportRouteGroupCacheEpoch === runtimeCacheEpoch && transportRouteGroupCacheKey === signature) return transportRouteGroupCache;
  const groups = new Map();
  (S.transportNetworks || []).forEach(route => {
    const key = getTransportRouteChainId(route);
    if(!key) return;
    if(!groups.has(key)) groups.set(key, []);
    groups.get(key).push(route);
  });
  transportRouteGroupCache = Array.from(groups.entries()).map(([chainId, routes]) => {
    const ordered = [...routes].sort((a, b) => {
      const aIdx = typeof a.segmentIndex === 'number' ? a.segmentIndex : 0;
      const bIdx = typeof b.segmentIndex === 'number' ? b.segmentIndex : 0;
      return aIdx - bIdx;
    });
    const first = ordered[0];
    const last = ordered[ordered.length - 1];
    const from = prefectureMap[first?.fromPrefectureId];
    const to = prefectureMap[last?.toPrefectureId];
    const activeRoutes = ordered.filter(route => route.status === 'active');
    const monthlyUsers = ordered.reduce((sum, route) => sum + (route.monthlyUsers || route.weeklyUsers || 0), 0);
    const monthlyRevenue = ordered.reduce((sum, route) => sum + (route.monthlyRevenue || route.weeklyRevenue || 0), 0);
    const monthlyBalance = ordered.reduce((sum, route) => sum + (route.monthlyBalance || route.weeklyBalance || 0), 0);
    const avgRate = ordered.length ? ordered.reduce((sum, route) => sum + (route.operatingRate || 0), 0) / ordered.length : 0;
    const buildingRoute = ordered.find(route => route.status === 'building');
    return {
      chainId,
      routes:ordered,
      type:first?.type || '',
      from,
      to,
      status:buildingRoute ? 'building' : 'active',
      weeksLeft:buildingRoute?.weeksLeft || 0,
      operatingRate:Math.round(avgRate),
      monthlyUsers,
      monthlyRevenue,
      monthlyBalance,
      activeCount:activeRoutes.length,
      segmentCount:ordered.length,
      label:first?.chainLabel || `${from?.name || '始点'} → ${to?.name || '終点'}`,
    };
  });
  transportRouteGroupCacheEpoch = runtimeCacheEpoch;
  transportRouteGroupCacheKey = signature;
  return transportRouteGroupCache;
}

function getTransportNetworkColor(type){
  const colors = {
    rail:'#8dd6ff',
    expressway:'#ffd180',
    ferry:'#7ce6c4',
    airlink:'#f8a5ff',
  };
  return colors[type] || '#d7f1ff';
}

function getTransportNetworkTravelWeeks(type){
  const config = transportNetworkCatalog[type];
  return Math.max(4, Math.round((config?.buildWeeks || 14) * 0.55));
}

function getPublicWorkDemandProfile(work){
  const config = publicWorksCatalog[work?.type];
  if(!config) return {users:0, revenue:0, balance:0};
  const scale = getPublicWorkScale(work);
  const regionPrefectures = prefectures.filter(prefecture => prefecture.region === work.region);
  const prefecturePopulation = regionPrefectures.reduce((sum, prefecture) => sum + (S.prefectureStats?.[prefecture.id]?.population || 0), 0);
  const economicMass = regionPrefectures.reduce((sum, prefecture) => sum + getPrefectureEconomicMass(prefecture.id), 0);
  const disasterPenalty = work.damage ? Math.max(0.38, 1 - work.damage * 0.6) : 1;
  const epidemicBoost = (S.activeMajorEvent?.id === 'pandemic' || S.activeMajorEvent?.id === 'bio_outbreak') ? 1.6 : 1;
  let users = 0;
  let revenue = 0;
  if(work.type === 'hospital'){
    users = Math.round((prefecturePopulation * 0.06 + economicMass * 18) * scale * disasterPenalty * epidemicBoost);
    revenue = users * 0.021 + (epidemicBoost > 1 ? users * 0.014 : 0);
  } else if(work.type === 'industrial'){
    users = Math.round((prefecturePopulation * 0.04 + economicMass * 42) * scale * disasterPenalty);
    revenue = users * 0.031 + Math.max(0, config.throughput || 0) * 1.02 * scale;
  } else {
    users = Math.round((prefecturePopulation * 0.03 + economicMass * 24) * scale * disasterPenalty);
    revenue = users * 0.0185 + Math.max(0, config.output || 0) * 0.88 * scale;
  }
  const upkeep = (config.upkeep || 0) * scale;
  return {
    users:Math.max(0, users),
    revenue:Math.max(0, revenue),
    balance:Math.max(-999999999, revenue - upkeep),
  };
}

function getPrefectureEconomicMass(prefectureId){
  const localCompanies = getPrefectureCompanies(prefectureId);
  if(!localCompanies.length) return 0;
  return localCompanies.reduce((sum, company) => sum + (company.revenue || 0), 0) / 1000;
}

const JAPAN_COMPANY_COORD_OVERRIDES = {
  aomori:{lat:40.1, lon:143.4, name:'青森フーズ'},
  akita:{lat:38.8, lon:142.9, name:'秋田デジタル'},
  iwate:{lat:38.6, lon:144.1, name:'岩手デジタル'},
  yamagata:{lat:37.4, lon:142.6, name:'山形ネット'},
  miyagi:{lat:36.8, lon:143.7, name:'宮城ネット'},
  fukushima:{lat:35.6, lon:143.1, name:'福島エナジー'},
  niigata:{lat:35.9, lon:141.4, name:'新潟メタル'},
  gunma:{lat:34.4, lon:141.4, name:'群馬県の会社'},
  tochigi:{lat:34.6, lon:142.4, name:'栃木'},
  ibaraki:{lat:34.0, lon:143.0, name:'茨城'},
  saitama:{lat:33.8, lon:141.9, name:'埼玉'},
  chiba:{lat:32.9, lon:142.9, name:'千葉'},
  tokyo:{lat:33.3, lon:142.3, name:'東京'},
  kanagawa:{lat:32.9, lon:141.8, name:'神奈川'},
  nagano:{lat:33.9, lon:140.2, name:'長野'},
  yamanashi:{lat:33.3, lon:141.0, name:'山梨'},
  shizuoka:{lat:32.3, lon:140.4, name:'静岡'},
  aichi:{lat:32.3, lon:139.2, name:'愛知'},
  gifu:{lat:33.3, lon:139.0, name:'岐阜'},
  toyama:{lat:34.7, lon:139.2, name:'富山'},
  ishikawa:{lat:35.5, lon:139.0, name:'石川'},
  fukui:{lat:33.7, lon:137.9, name:'福井'},
  shiga:{lat:32.6, lon:137.9, name:'滋賀'},
  mie:{lat:31.7, lon:138.6, name:'三重'},
  kyoto:{lat:32.6, lon:137.2, name:'京都'},
  nara:{lat:31.6, lon:137.6, name:'奈良'},
  wakayama:{lat:30.9, lon:136.9, name:'和歌山'},
  osaka:{lat:31.9, lon:137.1, name:'大阪'},
  hyogo:{lat:32.7, lon:136.2, name:'兵庫'},
  kagawa:{lat:31.5, lon:135.2, name:'香川'},
  tokushima:{lat:31.0, lon:136.0, name:'徳島'},
  kochi:{lat:30.1, lon:134.3, name:'高知'},
  ehime:{lat:30.6, lon:133.5, name:'愛媛'},
  tottori:{lat:33.1, lon:135.2, name:'鳥取'},
  shimane:{lat:32.7, lon:133.5, name:'島根'},
  okayama:{lat:32.0, lon:135.0, name:'岡山'},
  hiroshima:{lat:31.6, lon:133.6, name:'広島'},
  yamaguchi:{lat:31.2, lon:132.1, name:'山口'},
  fukuoka:{lat:30.9, lon:131.1, name:'福岡'},
  saga:{lat:30.1, lon:130.6, name:'佐賀'},
  nagasaki:{lat:29.7, lon:130.0, name:'長崎'},
  kagoshima:{lat:28.0, lon:131.0, name:'鹿児島'},
  kumamoto:{lat:29.5, lon:131.1, name:'熊本'},
  miyazaki:{lat:28.6, lon:132.1, name:'宮崎'},
  okinawa:{lat:20.6, lon:127.9, name:'沖縄'},
};

function getPrefectureCompanySpread(prefectureId, index){
  const profile = {
    tokyo:{biasX:-0.00115, biasY:0.00006, spreadX:0.00145, spreadY:0.00086},
    kanagawa:{biasX:-0.00062, biasY:0.00008, spreadX:0.00104, spreadY:0.00072},
    chiba:{biasX:-0.00092, biasY:0.00003, spreadX:0.0011, spreadY:0.00078},
    saitama:{biasX:-0.00058, biasY:-0.00004, spreadX:0.00102, spreadY:0.00068},
    ibaraki:{biasX:-0.00052, biasY:0.00002, spreadX:0.00096, spreadY:0.00066},
    gunma:{biasX:-0.0004, biasY:-0.00005, spreadX:0.00088, spreadY:0.00062},
    tochigi:{biasX:-0.00036, biasY:-0.00002, spreadX:0.0009, spreadY:0.00062},
    osaka:{biasX:-0.00042, biasY:0.00004, spreadX:0.00092, spreadY:0.00062},
    hyogo:{biasX:-0.00032, biasY:0.00004, spreadX:0.00094, spreadY:0.00066},
    kyoto:{biasX:-0.00028, biasY:-0.00002, spreadX:0.00084, spreadY:0.00058},
    hokkaido:{biasX:0.00008, biasY:0.00002, spreadX:0.00102, spreadY:0.00076},
    fukuoka:{biasX:-0.00022, biasY:-0.00002, spreadX:0.00078, spreadY:0.00056},
    saga:{biasX:-0.00018, biasY:0.00002, spreadX:0.00068, spreadY:0.0005},
    nagasaki:{biasX:-0.0003, biasY:0.00004, spreadX:0.00082, spreadY:0.00058},
    kagoshima:{biasX:-0.00012, biasY:0.00008, spreadX:0.00086, spreadY:0.00064},
    kumamoto:{biasX:-0.00012, biasY:0.00002, spreadX:0.00076, spreadY:0.00054},
    miyazaki:{biasX:0.00008, biasY:0.00002, spreadX:0.00076, spreadY:0.00054},
    okinawa:{biasX:0.00002, biasY:0.00002, spreadX:0.00052, spreadY:0.00042},
  }[prefectureId] || {biasX:0, biasY:0, spreadX:0.00062, spreadY:0.00048};
  const ring = Math.floor(index / 4);
  const slot = index % 4;
  const pattern = [
    {x:-0.76, y:-0.12},
    {x:-0.24, y:0.72},
    {x:0.42, y:-0.44},
    {x:0.86, y:0.18},
  ];
  const scale = 1 + ring * 0.42;
  return {
    dx:profile.biasX + pattern[slot].x * profile.spreadX * scale,
    dy:profile.biasY + pattern[slot].y * profile.spreadY * scale,
  };
}

function ensureCompanyMapPositions(){
  ensurePrefectureCompanyCoverage();
  const prefectureUsage = {};
  const fallbackUsage = {};
  companies.forEach(company => {
    const prefecture = prefectureMap[company.prefecture];
    if(prefecture){
      const override = JAPAN_COMPANY_COORD_OVERRIDES[prefecture.id];
      const useIndex = prefectureUsage[prefecture.id] || 0;
      const anchor = override ? geoToNormalized('japan', override.lat, override.lon) : {x:prefecture.x, y:prefecture.y};
      const spread = getPrefectureCompanySpread(prefecture.id, useIndex);
      const rawX = clamp(anchor.x + spread.dx, anchor.x - 0.0024, anchor.x + 0.0024);
      const rawY = clamp(anchor.y + spread.dy, anchor.y - 0.002, anchor.y + 0.002);
      const snapped = getReferenceMask('japan')
        ? snapPointToReferenceLand('japan', rawX, rawY, anchor.x, anchor.y, 0.008)
        : {x:rawX, y:rawY};
      company.mapX = snapped.x;
      company.mapY = snapped.y;
      company.mapLayoutVersion = 5;
      prefectureUsage[prefecture.id] = useIndex + 1;
      return;
    }
    const region = regions[company.region];
    if(!region || region.isSea) return;
    const useIndex = fallbackUsage[company.region] || 0;
    const angle = useIndex * 1.83 + 0.24;
    const orbit = 0.0014 + (useIndex % 4) * 0.00085;
    const rawX = clamp(region.cx + Math.cos(angle) * orbit, region.cx - 0.028, region.cx + 0.028);
    const rawY = clamp(region.cy + Math.sin(angle) * orbit, region.cy - 0.022, region.cy + 0.022);
    const regionSnapped = projectPointIntoRegionLand(rawX, rawY, company.region, region.cx, region.cy);
    const snapped = getReferenceMask('japan')
      ? snapPointToReferenceLand('japan', regionSnapped.x, regionSnapped.y, region.cx, region.cy, 0.016)
      : regionSnapped;
    company.mapX = snapped.x;
    company.mapY = snapped.y;
    company.mapLayoutVersion = 3;
    fallbackUsage[company.region] = useIndex + 1;
  });
  invalidateRuntimeCaches('company-map-layout');
}

// ============================================================
// TRADE PARTNERS
// ============================================================
const JAPAN_REFERENCE_GDP_USD = 4.0262;
function scalePartnerGdp(actualUsdTrillions){
  return Math.round(5400 * (actualUsdTrillions / JAPAN_REFERENCE_GDP_USD));
}

const tradePartners = [
  {id:'usa', name:'🇺🇸 アメリカ', sells:['oil','gas','cotton','iron','aluminum'], buys:['semiconductor','rice','tea','tech','space_goods','quantum_devices'], relation:80, cyberPower:124, nukePower:100, economy:128, gdpUsd:28.75, growth:2.8, inflation:2.9, unemployment:4.0, techLevel:82, stability:78, currencyCode:'USD', currencyName:'ドル', currencyRate:149.2, baseCurrencyRate:149.2, travelWeeks:6},
  {id:'china', name:'🇨🇳 中国', sells:['rareearth','iron','coal','cotton','rubber','aluminum'], buys:['semiconductor','silicon','fish','auto','advanced_materials'], relation:45, cyberPower:118, nukePower:80, economy:118, gdpUsd:18.7438, growth:5.0, inflation:0.2, unemployment:4.6, techLevel:78, stability:61, currencyCode:'CNY', currencyName:'人民元', currencyRate:20.6, baseCurrencyRate:20.6, travelWeeks:4},
  {id:'australia', name:'🇦🇺 オーストラリア', sells:['iron','coal','gas','uranium','gold'], buys:['rice','semiconductor','tea','medical_goods'], relation:78, cyberPower:74, nukePower:0, economy:82, gdpUsd:1.88, growth:1.9, inflation:3.8, unemployment:4.2, techLevel:63, stability:77, currencyCode:'AUD', currencyName:'豪ドル', currencyRate:95.0, baseCurrencyRate:95.0, travelWeeks:5},
  {id:'saudi', name:'🇸🇦 サウジアラビア', sells:['oil','gas'], buys:['semiconductor','rice','tech','energy_cells'], relation:68, cyberPower:70, nukePower:0, economy:76, gdpUsd:1.24, growth:1.8, inflation:1.7, unemployment:3.9, techLevel:56, stability:72, currencyCode:'SAR', currencyName:'リヤル', currencyRate:39.7, baseCurrencyRate:39.7, travelWeeks:6},
  {id:'germany', name:'🇩🇪 ドイツ', sells:['iron','coal','aluminum'], buys:['semiconductor','silicon','auto','tech','advanced_materials'], relation:72, cyberPower:108, nukePower:0, economy:96, gdpUsd:4.66, growth:0.2, inflation:2.2, unemployment:3.4, techLevel:75, stability:76, currencyCode:'EUR', currencyName:'ユーロ', currencyRate:161.5, baseCurrencyRate:161.5, travelWeeks:7},
  {id:'brazil', name:'🇧🇷 ブラジル', sells:['iron','rubber','cotton'], buys:['semiconductor','silicon','medical_goods'], relation:58, cyberPower:72, nukePower:0, economy:74, gdpUsd:2.19, growth:3.4, inflation:4.4, unemployment:6.0, techLevel:55, stability:63, currencyCode:'BRL', currencyName:'レアル', currencyRate:26.2, baseCurrencyRate:26.2, travelWeeks:8},
  {id:'chile', name:'🇨🇱 チリ', sells:['copper','lithium'], buys:['rice','fish','energy_cells'], relation:62, cyberPower:66, nukePower:0, economy:58, gdpUsd:0.33027, growth:2.6, inflation:4.3, unemployment:8.7, techLevel:48, stability:62, currencyCode:'CLP', currencyName:'チリ・ペソ', currencyRate:0.16, baseCurrencyRate:0.16, travelWeeks:9},
  {id:'indonesia', name:'🇮🇩 インドネシア', sells:['rubber','gas','coal','gold'], buys:['semiconductor','rice','food'], relation:64, cyberPower:82, nukePower:0, economy:79, gdpUsd:1.4, growth:5.0, inflation:2.2, unemployment:3.3, techLevel:58, stability:68, currencyCode:'IDR', currencyName:'ルピア', currencyRate:0.0093, baseCurrencyRate:0.0093, travelWeeks:6},
  {id:'korea', name:'🇰🇷 韓国', sells:['silicon','semiconductor'], buys:['fish','timber','gas','tech','quantum_devices'], relation:60, cyberPower:112, nukePower:0, economy:92, gdpUsd:1.88, growth:2.0, inflation:2.3, unemployment:2.7, techLevel:76, stability:74, currencyCode:'KRW', currencyName:'ウォン', currencyRate:0.11, baseCurrencyRate:0.11, travelWeeks:4},
  {id:'india', name:'🇮🇳 インド', sells:['iron','cotton','rareearth','titanium'], buys:['semiconductor','silicon','auto','tech','medical_goods'], relation:62, cyberPower:104, nukePower:40, economy:104, gdpUsd:3.9127, growth:6.5, inflation:5.0, unemployment:4.2, techLevel:69, stability:69, currencyCode:'INR', currencyName:'ルピー', currencyRate:1.76, baseCurrencyRate:1.76, travelWeeks:7},
  {id:'uk', name:'🇬🇧 イギリス', sells:['gas','oil','gold'], buys:['semiconductor','tech','medical_goods','quantum_devices','space_goods'], relation:76, cyberPower:116, nukePower:55, economy:98, gdpUsd:3.59, growth:1.1, inflation:2.8, unemployment:4.3, techLevel:79, stability:77, currencyCode:'GBP', currencyName:'ポンド', currencyRate:188.4, baseCurrencyRate:188.4, travelWeeks:7},
  {id:'france', name:'🇫🇷 フランス', sells:['uranium','gas','aluminum','gold'], buys:['semiconductor','auto','tech','medical_goods','advanced_materials'], relation:74, cyberPower:103, nukePower:42, economy:94, gdpUsd:3.13, growth:1.0, inflation:2.6, unemployment:7.1, techLevel:73, stability:74, currencyCode:'EUR', currencyName:'ユーロ', currencyRate:161.5, baseCurrencyRate:161.5, travelWeeks:7},
  {id:'canada', name:'🇨🇦 カナダ', sells:['oil','gas','timber','uranium','gold'], buys:['semiconductor','tech','auto','medical_goods'], relation:79, cyberPower:90, nukePower:0, economy:87, gdpUsd:2.24, growth:1.4, inflation:2.7, unemployment:5.9, techLevel:68, stability:79, currencyCode:'CAD', currencyName:'カナダドル', currencyRate:109.8, baseCurrencyRate:109.8, travelWeeks:8},
  {id:'mexico', name:'🇲🇽 メキシコ', sells:['oil','gas','copper','cotton'], buys:['auto','semiconductor','tech','food','medical_goods'], relation:57, cyberPower:80, nukePower:0, economy:83, gdpUsd:1.9, growth:2.2, inflation:4.1, unemployment:3.0, techLevel:60, stability:64, currencyCode:'MXN', currencyName:'メキシコペソ', currencyRate:8.8, baseCurrencyRate:8.8, travelWeeks:8},
  {id:'uae', name:'🇦🇪 UAE', sells:['oil','gas','gold'], buys:['semiconductor','tech','medical_goods','energy_cells','space_goods'], relation:71, cyberPower:88, nukePower:0, economy:81, gdpUsd:0.54, growth:3.8, inflation:2.1, unemployment:2.6, techLevel:64, stability:78, currencyCode:'AED', currencyName:'ディルハム', currencyRate:40.6, baseCurrencyRate:40.6, travelWeeks:6},
  {id:'vietnam', name:'🇻🇳 ベトナム', sells:['rubber','coal','fish','cotton'], buys:['semiconductor','tech','medical_goods','advanced_materials'], relation:66, cyberPower:86, nukePower:0, economy:76, gdpUsd:0.47, growth:5.8, inflation:3.3, unemployment:2.2, techLevel:59, stability:72, currencyCode:'VND', currencyName:'ドン', currencyRate:0.0062, baseCurrencyRate:0.0062, travelWeeks:5},
  {id:'turkey', name:'🇹🇷 トルコ', sells:['coal','cotton','gold','iron'], buys:['auto','tech','medical_goods','energy_cells'], relation:52, cyberPower:85, nukePower:0, economy:80, gdpUsd:1.12, growth:3.2, inflation:41.0, unemployment:8.7, techLevel:60, stability:58, currencyCode:'TRY', currencyName:'リラ', currencyRate:4.3, baseCurrencyRate:4.3, travelWeeks:7},
  {id:'southafrica', name:'🇿🇦 南アフリカ', sells:['gold','coal','iron','uranium'], buys:['tech','auto','medical_goods','energy_cells'], relation:55, cyberPower:74, nukePower:0, economy:69, gdpUsd:0.41, growth:1.6, inflation:5.1, unemployment:32.0, techLevel:52, stability:54, currencyCode:'ZAR', currencyName:'ランド', currencyRate:7.9, baseCurrencyRate:7.9, travelWeeks:9},
];

tradePartners.forEach((partner, idx) => {
  partner.gdp = scalePartnerGdp(partner.gdpUsd);
  if(typeof partner.techLevel !== 'number') partner.techLevel = 20 + partner.cyberPower * 0.45;
  if(typeof partner.stability !== 'number') partner.stability = 48 + partner.relation * 0.25;
  if(typeof partner.health !== 'number') partner.health = 52 + ((idx * 7) % 28);
  if(typeof partner.hostility !== 'number') partner.hostility = Math.max(0, 100 - partner.relation);
  if(typeof partner.nuclearScars !== 'number') partner.nuclearScars = 0;
  if(typeof partner.spyIntelWeeks !== 'number') partner.spyIntelWeeks = 0;
  if(typeof partner.spyAlert !== 'number') partner.spyAlert = 0;
  if(!Array.isArray(partner.currencyHist)) partner.currencyHist = Array.from({length:1040}, () => partner.currencyRate * rand(0.985, 1.018));
});
const partnerPopulations = {usa:335, china:1410, australia:27, saudi:37, germany:84, brazil:216, chile:20, indonesia:281, korea:52, india:1430, uk:68, france:65, canada:41, mexico:130, uae:10, vietnam:101, turkey:86, southafrica:63};
tradePartners.forEach(partner => {
  if(typeof partner.population !== 'number') partner.population = partnerPopulations[partner.id] || 50;
});

const worldBackdropPolys = [
  [[.05,.18],[.24,.16],[.31,.24],[.28,.41],[.17,.43],[.08,.34]],
  [[.22,.48],[.31,.49],[.35,.66],[.31,.86],[.24,.92],[.18,.82],[.17,.63]],
  [[.36,.21],[.66,.18],[.79,.27],[.9,.33],[.93,.46],[.79,.55],[.67,.57],[.55,.48],[.45,.46],[.38,.34]],
  [[.46,.47],[.57,.5],[.61,.67],[.53,.84],[.43,.74],[.42,.57]],
  [[.74,.64],[.86,.65],[.93,.76],[.9,.87],[.8,.9],[.72,.8]],
];

const worldCountries = [
  {
    id:'japan',
    name:'🇯🇵 日本',
    label:'日本',
    cx:.894, cy:.364,
    lat:35.68, lon:139.76,
    chipDx:28,
    chipDy:-20,
    hitRadius:26,
    displaySectors:['tech','auto','finance'],
    majorCompanies:['トヨダ自動車','ソニック・グループ','三星UFJ銀行'],
    note:'国家経営の本拠地。世界マップでは各国との関係と交易線の中心になります。'
  },
  {
    id:'usa',
    partnerIdx:0,
    name:'🇺🇸 アメリカ',
    label:'アメリカ',
    cx:.192, cy:.348,
    lat:38.90, lon:-77.04,
    chipDx:-10,
    chipDy:-26,
    hitRadius:28,
    displaySectors:['tech','defense','energy'],
    majorCompanies:['Aegis Dynamics','Pacific Nexus','Frontier Energy'],
    note:'テック、防衛、エネルギーの厚みがあり、世界景気の主導役になりやすい国です。'
  },
  {
    id:'china',
    partnerIdx:1,
    name:'🇨🇳 中国',
    label:'中国',
    cx:.776, cy:.378,
    lat:39.90, lon:116.40,
    chipDx:-54,
    chipDy:-18,
    hitRadius:26,
    displaySectors:['mining','tech','shipping'],
    majorCompanies:['Sino Materials','Dragon Port','Pearl Compute'],
    note:'レアアースと製造力が強く、供給網イベントの中心になりやすい相手です。'
  },
  {
    id:'australia',
    partnerIdx:2,
    name:'🇦🇺 オーストラリア',
    label:'豪州',
    cx:.866, cy:.796,
    lat:-35.28, lon:149.13,
    chipDx:0,
    chipDy:24,
    hitRadius:28,
    displaySectors:['mining','energy','food'],
    majorCompanies:['Outback Minerals','Coral LNG','Southern Agri'],
    note:'鉱石・LNG・食料の安定供給源として使いやすい友好国です。'
  },
  {
    id:'saudi',
    partnerIdx:3,
    name:'🇸🇦 サウジアラビア',
    label:'サウジ',
    cx:.594, cy:.462,
    lat:24.71, lon:46.67,
    chipDx:34,
    chipDy:8,
    hitRadius:24,
    displaySectors:['energy','finance','shipping'],
    majorCompanies:['Desert Petro','Gulf Terminal','Vision Capital'],
    note:'原油供給の柱。エネルギー危機のときに戦略的重要度が跳ね上がります。'
  },
  {
    id:'germany',
    partnerIdx:4,
    name:'🇩🇪 ドイツ',
    label:'ドイツ',
    cx:.505, cy:.312,
    lat:52.52, lon:13.40,
    chipDx:0,
    chipDy:-28,
    hitRadius:22,
    displaySectors:['tech','construction','auto'],
    majorCompanies:['Rhine Systems','Autobahn Werke','Europa Steel'],
    note:'工業・機械・インフラが強く、欧州の景気感応度が高い国です。'
  },
  {
    id:'brazil',
    partnerIdx:5,
    name:'🇧🇷 ブラジル',
    label:'ブラジル',
    cx:.241, cy:.71,
    lat:-15.79, lon:-47.88,
    chipDx:42,
    chipDy:-4,
    hitRadius:26,
    displaySectors:['food','mining','energy'],
    majorCompanies:['Amazon Agro','Verde Metals','Atlântico Power'],
    note:'農産物・鉱物・内需のバランスがあり、中期の資源調達先として優秀です。'
  },
  {
    id:'chile',
    partnerIdx:6,
    name:'🇨🇱 チリ',
    label:'チリ',
    cx:.21, cy:.834,
    lat:-33.45, lon:-70.67,
    chipDx:-14,
    chipDy:20,
    hitRadius:20,
    displaySectors:['mining','energy','shipping'],
    majorCompanies:['Andes Lithium','Pacific Copper','Atacama Ports'],
    note:'銅とリチウムの要衝。電池・自動車・半導体系の交渉先として重要です。'
  },
  {
    id:'indonesia',
    partnerIdx:7,
    name:'🇮🇩 インドネシア',
    label:'インドネシア',
    cx:.8, cy:.624,
    lat:-6.21, lon:106.85,
    chipDx:28,
    chipDy:16,
    hitRadius:26,
    displaySectors:['energy','food','shipping'],
    majorCompanies:['Java LNG','Archipelago Rubber','Strait Logistics'],
    note:'ゴム・ガス・海上物流の結節点。海運や製造業の調整弁になります。'
  },
  {
    id:'korea',
    partnerIdx:8,
    name:'🇰🇷 韓国',
    label:'韓国',
    cx:.842, cy:.362,
    lat:37.57, lon:126.98,
    chipDx:-26,
    chipDy:-22,
    hitRadius:20,
    displaySectors:['tech','telecom','shipping'],
    majorCompanies:['Han Compute','Blue Wave Telecom','K-Sea Logistics'],
    note:'半導体と通信が強く、日本のITサプライチェーンと近接して動きます。'
  },
  {
    id:'india',
    partnerIdx:9,
    name:'🇮🇳 インド',
    label:'インド',
    cx:.695, cy:.504,
    lat:28.61, lon:77.21,
    chipDx:-40,
    chipDy:14,
    hitRadius:25,
    displaySectors:['tech','construction','mining'],
    majorCompanies:['Bharat Infra','Lotus Compute','Deccan Metals'],
    note:'人口と成長率の勢いが強く、中長期の輸出先・生産提携先として伸びやすい国です。'
  },
  {
    id:'uk',
    partnerIdx:10,
    name:'🇬🇧 イギリス',
    label:'イギリス',
    cx:.47, cy:.288,
    lat:51.51, lon:-0.13,
    chipDx:-38,
    chipDy:-26,
    hitRadius:22,
    displaySectors:['finance','tech','defense'],
    majorCompanies:['Crown Quantum','Albion Defense','North Sea Capital'],
    note:'金融と防衛研究の厚みがあり、欧州圏のリスクマネーと軍需の両面で存在感を持つ国です。'
  },
  {
    id:'france',
    partnerIdx:11,
    name:'🇫🇷 フランス',
    label:'フランス',
    cx:.495, cy:.334,
    lat:48.86, lon:2.35,
    chipDx:34,
    chipDy:4,
    hitRadius:22,
    displaySectors:['construction','energy','space'],
    majorCompanies:['Gaulle Energies','Orbital Europe','Hexa Mobility'],
    note:'原子力・宇宙・インフラが強く、欧州の産業政策イベントに絡みやすい国です。'
  },
  {
    id:'canada',
    partnerIdx:12,
    name:'🇨🇦 カナダ',
    label:'カナダ',
    cx:.174, cy:.246,
    lat:45.42, lon:-75.69,
    chipDx:-8,
    chipDy:-28,
    hitRadius:26,
    displaySectors:['mining','energy','food'],
    majorCompanies:['Maple Uranium','Polar LNG','Prairie Agro'],
    note:'資源と農産物流通が安定しており、長期契約向きの北米パートナーです。'
  },
  {
    id:'mexico',
    partnerIdx:13,
    name:'🇲🇽 メキシコ',
    label:'メキシコ',
    cx:.151, cy:.454,
    lat:19.43, lon:-99.13,
    chipDx:26,
    chipDy:18,
    hitRadius:22,
    displaySectors:['auto','mining','shipping'],
    majorCompanies:['Azteca Auto','Sierra Copper','Pacific Border Port'],
    note:'製造輸出と資源の中継地。北米サプライチェーン再編の受け皿になりやすい国です。'
  },
  {
    id:'uae',
    partnerIdx:14,
    name:'🇦🇪 UAE',
    label:'UAE',
    cx:.618, cy:.468,
    lat:24.45, lon:54.38,
    chipDx:44,
    chipDy:-10,
    hitRadius:20,
    displaySectors:['energy','finance','space'],
    majorCompanies:['Desert Launch','Emirates Capital','Gulf Energy Hub'],
    note:'資本力とエネルギー輸出に加えて宇宙投資も強く、短期の資金波及が大きい国です。'
  },
  {
    id:'vietnam',
    partnerIdx:15,
    name:'🇻🇳 ベトナム',
    label:'ベトナム',
    cx:.776, cy:.464,
    lat:21.03, lon:105.85,
    chipDx:34,
    chipDy:-8,
    hitRadius:21,
    displaySectors:['shipping','tech','food'],
    majorCompanies:['Saigon Circuits','Red River Ports','Mekong Foods'],
    note:'海運と組立産業が伸びやすく、中国代替の供給網先として機能しやすい国です。'
  },
  {
    id:'turkey',
    partnerIdx:16,
    name:'🇹🇷 トルコ',
    label:'トルコ',
    cx:.565, cy:.389,
    lat:39.93, lon:32.85,
    chipDx:34,
    chipDy:-16,
    hitRadius:22,
    displaySectors:['construction','shipping','defense'],
    majorCompanies:['Anatolia Steel','Bosporus Defense','Crossroad Logistics'],
    note:'欧州と中東をつなぐ結節点。物流や軍需の政治イベントで振れ幅が出やすい国です。'
  },
  {
    id:'southafrica',
    partnerIdx:17,
    name:'🇿🇦 南アフリカ',
    label:'南アフリカ',
    cx:.552, cy:.804,
    lat:-25.75, lon:28.19,
    chipDx:36,
    chipDy:12,
    hitRadius:22,
    displaySectors:['mining','energy','shipping'],
    majorCompanies:['Cape Minerals','Savanna Grid','Good Hope Port'],
    note:'鉱物と港湾の存在感が強く、資源価格イベントに反応しやすいアフリカ南部の要衝です。'
  },
];

const WORLD_COUNTRY_UI_OVERRIDES = {
  japan:{gridV:17.5, gridH:145.9, lat:17.5, lon:145.9, chipDx:26, chipDy:-18, nudgeX:-0.012, nudgeY:-0.012},
  korea:{gridV:38.4, gridH:138.3, lat:38.4, lon:138.3, chipDx:-30, chipDy:-26, nudgeX:-0.01, nudgeY:-0.006},
  china:{gridV:28.8, gridH:127.0, lat:28.8, lon:127.0, chipDx:-46, chipDy:-12, nudgeX:-0.02, nudgeY:-0.004},
  indonesia:{gridV:6.7, gridH:117.3, lat:-6.7, lon:117.3, chipDx:34, chipDy:18, nudgeX:0.014, nudgeY:-0.005},
  australia:{gridV:37.1, gridH:169.3, lat:-37.1, lon:169.3, chipDx:8, chipDy:26, nudgeX:-0.02, nudgeY:-0.01},
  india:{gridV:15.2, gridH:80.9, lat:15.2, lon:80.9, chipDx:-38, chipDy:14, nudgeX:0.004, nudgeY:-0.01},
  saudi:{gridV:23.9, gridH:44.0, lat:23.9, lon:44.0, chipDx:36, chipDy:6, nudgeX:0.004, nudgeY:-0.004},
  brazil:{gridV:20.9, gridH:80.6, lat:-20.9, lon:-80.6, chipDx:40, chipDy:-4, nudgeX:-0.008, nudgeY:-0.004},
  chile:{gridV:36.9, gridH:101.4, lat:-36.9, lon:-101.4, chipDx:-16, chipDy:18, nudgeX:0.004, nudgeY:-0.008},
  germany:{gridV:52.2, gridH:0.5, lat:52.2, lon:0.5, chipDx:4, chipDy:-28, nudgeX:0, nudgeY:0},
  uk:{gridV:51.5, gridH:0.1, lat:51.5, lon:-0.1, chipDx:-38, chipDy:-28, nudgeX:-0.002, nudgeY:-0.004},
  france:{gridV:48.9, gridH:2.3, lat:48.9, lon:2.3, chipDx:36, chipDy:6, nudgeX:0.003, nudgeY:0.004},
  canada:{gridV:45.4, gridH:75.7, lat:45.4, lon:-75.7, chipDx:-10, chipDy:-28, nudgeX:-0.01, nudgeY:-0.02},
  mexico:{gridV:19.4, gridH:99.1, lat:19.4, lon:-99.1, chipDx:28, chipDy:18, nudgeX:0.006, nudgeY:0.004},
  uae:{gridV:24.5, gridH:54.4, lat:24.5, lon:54.4, chipDx:46, chipDy:-12, nudgeX:0.008, nudgeY:0.002},
  vietnam:{gridV:21.0, gridH:105.9, lat:21.0, lon:105.9, chipDx:36, chipDy:-8, nudgeX:0.014, nudgeY:0.004},
  turkey:{gridV:39.9, gridH:32.9, lat:39.9, lon:32.9, chipDx:34, chipDy:-16, nudgeX:0.004, nudgeY:0},
  southafrica:{gridV:25.8, gridH:28.2, lat:-25.8, lon:28.2, chipDx:38, chipDy:12, nudgeX:0.012, nudgeY:0.006},
};

function applyWorldCountryUiOverrides(){
  worldCountries.forEach(country => {
    const override = WORLD_COUNTRY_UI_OVERRIDES[country.id];
    if(!override) return;
    if(typeof override.lat === 'number') country.lat = override.lat;
    if(typeof override.lon === 'number') country.lon = override.lon;
    const mapped = geoToNormalized('world', country.lat, country.lon);
    country.cx = clamp((override.cx ?? mapped.x) + (override.nudgeX || 0), 0.02, 0.98);
    country.cy = clamp((override.cy ?? mapped.y) + (override.nudgeY || 0), 0.04, 0.96);
    country.gridV = override.gridV;
    country.gridH = override.gridH;
    if(typeof override.chipDx === 'number') country.chipDx = override.chipDx;
    if(typeof override.chipDy === 'number') country.chipDy = override.chipDy;
  });
}

applyWorldCountryUiOverrides();

function createForeignCompanies(){
  return worldCountries
    .filter(country => typeof country.partnerIdx === 'number')
    .flatMap(country => {
      const partner = tradePartners[country.partnerIdx];
      return (country.displaySectors || []).slice(0, 3).map((sectorId, index) => {
        const sector = sectors[sectorId] || sectors.tech;
        const gdpScale = Math.max(0.4, (partner?.gdpUsd || 1) / JAPAN_REFERENCE_GDP_USD);
        const basePrice = Math.max(120, Math.round(420 + gdpScale * 680 + index * 190 + (partner?.techLevel || 40) * 5));
        const sharesOutstanding = Math.max(8000, Math.round(12000 + gdpScale * 7000 + index * 900));
        const baseRevenue = Math.max(1600, Math.round(basePrice * sharesOutstanding * rand(0.015, 0.028)));
        return {
          id:`f-${country.id}-${index}`,
          countryId:country.id,
          countryName:country.name,
          partnerIdx:country.partnerIdx,
          name:`${country.label}${sector.name}${index + 1}`,
          sector:sectorId,
          price:basePrice,
          prevPrice:basePrice,
          base:basePrice,
          sharesOutstanding,
          revenue:baseRevenue,
          prevRevenue:baseRevenue,
          revenueHist:[baseRevenue],
          hist:[basePrice],
          margin:rand(0.05, 0.16),
          volatility:rand(0.006, 0.018),
          desc:`${country.name} の ${sector.name} 主要企業。戦争で勝利した国に限り投資可能。`,
        };
      });
    });
}

function ensureForeignCompanies(){
  if(!Array.isArray(S.foreignCompanies) || !S.foreignCompanies.length) S.foreignCompanies = createForeignCompanies();
  S.foreignCompanies.forEach(company => {
    if(!Array.isArray(company.hist)) company.hist = [company.price];
    if(!Array.isArray(company.revenueHist)) company.revenueHist = [company.revenue];
    if(typeof company.margin !== 'number') company.margin = 0.1;
    if(typeof company.volatility !== 'number') company.volatility = 0.012;
    if(typeof company.sharesOutstanding !== 'number') company.sharesOutstanding = 10000;
    if(typeof company.floorBase !== 'number') company.floorBase = Math.max(100, Math.round((company.base || company.price || 100) * rand(0.36, 0.62)));
    if(typeof company.softCeilingMult !== 'number') company.softCeilingMult = rand(1.85, 2.35);
    if(typeof company.overheatCooldown !== 'number') company.overheatCooldown = 0;
    if(typeof company.floorWaveSeed !== 'number') company.floorWaveSeed = rand(0, Math.PI * 2);
    if(typeof company.idleBias !== 'number') company.idleBias = rand(-0.28, 0.28);
    if(typeof company.revenueScaleHint !== 'number'){
      const sectorRevenueScale = {tech:0.022, telecom:0.021, defense:0.02, space:0.024, finance:0.017, hospital:0.016, pharma:0.019, construction:0.015, energy:0.017, auto:0.018, retail:0.014, food:0.013, mining:0.016, shipping:0.015};
      company.revenueScaleHint = (sectorRevenueScale[company.sector] || 0.016) * rand(0.9, 1.12);
    }
    if(typeof company.favorability !== 'number') company.favorability = 50;
    company.price = Math.max(100, company.price || company.base || 100);
  });
}

function canInvestInCountry(countryId){
  return S.warVictories.includes(countryId);
}

function getCountryShippingWeeks(partnerIdx){
  const partner = tradePartners[partnerIdx];
  return Math.max(3, Math.round(partner?.travelWeeks || 6));
}

function getBudgetSupportEntries(plan=S.annualBudget){
  return normalizeBudgetSupportList(plan?.companySupports, plan?.company, plan?.companySupport).filter(entry => entry.company && entry.amount > 0);
}

function getCompanyAnnualSupport(companyName, plan=S.annualBudget){
  return getBudgetSupportEntries(plan).filter(entry => entry.company === companyName).reduce((sum, entry) => sum + entry.amount, 0);
}

function getRoadmapLevel(company){
  return clamp(company.roadmapIndex || 0, 0, (company.roadmap || []).length);
}

function getCompanyTrack(company, trackId=company?.selectedTechTrack){
  if(!company) return null;
  syncCompanyRoadmap(company);
  if(!trackId) return null;
  return company.techTracks.find(track => track.id === trackId) || null;
}

window.selectCompanyTechTrack = function(companyName, trackId){
  const company = getCompanyByName(companyName);
  if(!company) return;
  syncCompanyRoadmap(company);
  const nextTrack = getCompanyTrack(company, trackId);
  const currentTrackId = company.selectedTechTrack || '';
  const nextTrackId = nextTrack?.id || '';
  if(currentTrackId === nextTrackId) return;
  persistCompanyRoadmapState(company, company.loadedRoadmapTrackKey || getRoadmapStateKey(currentTrackId));
  company.selectedTechTrack = nextTrackId;
  syncCompanyRoadmap(company);
  addEvent(`🧭 ${company.name} の技術方針を「${nextTrack ? nextTrack.name : '標準開発'}」へ変更`, '');
  openCompanyDetail(company.name);
};

window.setFocusedCompanyTechTrack = function(companyName, trackId){
  const company = getCompanyByName(companyName);
  if(!company) return;
  syncCompanyRoadmap(company);
  const nextTrack = getCompanyTrack(company, trackId);
  const currentTrackId = company.selectedTechTrack || '';
  const nextTrackId = nextTrack?.id || '';
  if(currentTrackId === nextTrackId) return;
  persistCompanyRoadmapState(company, company.loadedRoadmapTrackKey || getRoadmapStateKey(currentTrackId));
  company.selectedTechTrack = nextTrackId;
  S.techFocusCompany = company.name;
  syncCompanyRoadmap(company);
  addEvent(`🧭 ${company.name} の技術方針を「${nextTrack ? nextTrack.name : '標準開発'}」へ変更`, '');
  renderPanel();
  updateUI();
};

function getCompanyDynamicFloor(company){
  const floorBase = Math.max(100, company.floorBase || company.base || 100);
  const floorWave = Math.sin(S.totalWeeks * 0.14 + (company.floorWaveSeed || 0)) * floorBase * 0.035;
  const revenueSupport = clamp((company.revenue || 0) / Math.max(1, getHistoricReference(company.revenueHist, 52)) - 1, -0.18, 0.28);
  const favorSupport = ((company.favorability || 50) - 50) * floorBase * 0.0016;
  const floorLift = Math.max(0, (company.base || floorBase) - floorBase) * 0.18;
  return Math.max(100, Math.round(floorBase + floorWave + favorSupport + floorLift + floorBase * revenueSupport * 0.08));
}

function getCompanySoftCeiling(company){
  const dynamicFloor = getCompanyDynamicFloor(company);
  const growthLift = clamp((company.base || dynamicFloor) / Math.max(100, company.floorBase || dynamicFloor) - 1, 0, 1.6);
  return Math.max(dynamicFloor + 30, Math.round((company.base || dynamicFloor) * ((company.softCeilingMult || 2.1) + growthLift * 0.08)));
}

function maybeTriggerStockBaseShock(company, options={}){
  company.overheatCooldown = Math.max(0, (company.overheatCooldown || 0) - 1);
  const originalRatio = company.price / Math.max(100, company.floorBase || company.base || 100);
  const baseRatio = company.base / Math.max(100, company.floorBase || company.base || 100);
  const triggerRatio = options.triggerRatio || 2.55;
  if(company.overheatCooldown > 0 || (originalRatio < triggerRatio && baseRatio < triggerRatio * 0.84)) return false;
  const dynamicFloor = getCompanyDynamicFloor(company);
  company.overheatCooldown = 78;
  company.recoveryLag = Math.max(company.recoveryLag || 0, Math.round(rand(42, 96)));
  company.base = Math.max(dynamicFloor, Math.round(company.base * rand(0.52, 0.78)));
  company.price = Math.max(dynamicFloor, Math.round(company.price * rand(0.18, 0.68)));
  company.sentiment = clamp((company.sentiment || 0) - rand(0.18, 0.36), -0.8, 0.4);
  if(options.silent){
    return true;
  }
  addEvent(`📉 ${company.name} に過熱修正が発生。期待が剥がれ、基準値そのものが大きく切り下がった`, 'bad');
  return true;
}

function triangleWave(t){
  const wrapped = ((t % 1) + 1) % 1;
  return 1 - 4 * Math.abs(wrapped - 0.5);
}

function ensureRevenueWaveState(company){
  if(typeof company.revenueWaveSeed !== 'number') company.revenueWaveSeed = rand(0, Math.PI * 2);
  if(typeof company.revenueWaveRate !== 'number') company.revenueWaveRate = rand(0.08, 0.19);
  if(typeof company.revenueWaveAmp !== 'number') company.revenueWaveAmp = rand(0.016, 0.052);
  if(typeof company.revenueWaveDrift !== 'number') company.revenueWaveDrift = rand(-0.1, 0.1);
  if(typeof company.revenuePatternMode !== 'string') company.revenuePatternMode = pick(['range','swell','dip','plateau','rebound','plateau-drop']);
  if(typeof company.revenuePatternAge !== 'number') company.revenuePatternAge = Math.floor(rand(0, 8));
  if(typeof company.revenuePatternLength !== 'number') company.revenuePatternLength = Math.floor(rand(8, 22));
  if(typeof company.revenuePatternCompression !== 'number') company.revenuePatternCompression = rand(0.58, 1.14);
  if(typeof company.revenuePatternBias !== 'number') company.revenuePatternBias = rand(-0.35, 0.35);
  if(typeof company.revenuePatternPhaseSeed !== 'number') company.revenuePatternPhaseSeed = rand(0, Math.PI * 2);
}

function maybeRotateRevenuePattern(company, signal){
  ensureRevenueWaveState(company);
  company.revenuePatternAge = (company.revenuePatternAge || 0) + 1;
  if(company.revenuePatternAge < Math.max(6, company.revenuePatternLength || 10) && Math.random() >= 0.06 + Math.abs(signal) * 0.018) return;
  company.revenuePatternAge = 0;
  company.revenuePatternLength = Math.round(rand(8, 24));
  company.revenuePatternCompression = rand(0.54, 1.12);
  company.revenuePatternBias = clamp(signal * 0.68 + rand(-0.24, 0.24), -0.72, 0.72);
  const positivePool = ['range','swell','rebound','plateau'];
  const negativePool = ['range','dip','plateau-drop'];
  const neutralPool = ['range','swell','dip','plateau','rebound','plateau-drop'];
  company.revenuePatternMode = pick(signal > 0.08 ? positivePool : signal < -0.08 ? negativePool : neutralPool);
}

function getCompanyRevenuePatternForce(company){
  ensureRevenueWaveState(company);
  const age = (company.revenuePatternAge || 0) / Math.max(1, company.revenuePatternLength || 10);
  const primary = triangleWave(age + (company.revenuePatternPhaseSeed || 0) * 0.03);
  const micro = (((company.revenuePatternAge || 0) % 3) - 1) * 0.18;
  let shape = primary * (company.revenuePatternCompression || 0.9) + micro;
  let bias = company.revenuePatternBias || 0;
  switch(company.revenuePatternMode){
    case 'swell':
      shape *= 0.72;
      bias += 0.08 + age * 0.32;
      break;
    case 'dip':
      shape *= 0.72;
      bias -= 0.08 + age * 0.34;
      break;
    case 'plateau':
      shape *= 0.42;
      bias += 0.05;
      break;
    case 'rebound':
      shape *= 0.9;
      bias += -0.1 + age * 0.7;
      break;
    case 'plateau-drop':
      shape *= 0.45;
      bias += 0.16 - age * 0.82;
      break;
    default:
      bias *= 0.34;
      break;
  }
  return {shape, bias, age};
}

function maybeRotateCompanyPattern(company, signal){
  company.patternAge = (company.patternAge || 0) + 1;
  if(company.patternAge < Math.max(4, company.patternLength || 8) && Math.random() >= 0.045 + Math.abs(signal) * 0.02) return;
  company.patternAge = 0;
  company.patternLength = Math.round(rand(6, 16));
  company.patternCompression = rand(0.42, 1.08);
  company.patternBias = clamp(signal * 0.7 + rand(-0.32, 0.32), -0.9, 0.9);
  const positivePool = ['range','triangle-up','flag-up','bottoming'];
  const negativePool = ['range','triangle-down','flag-down','topping'];
  const neutralPool = ['range','triangle-up','triangle-down','bottoming','topping'];
  company.patternMode = pick(signal > 0.08 ? positivePool : signal < -0.08 ? negativePool : neutralPool);
}

function getCompanyPatternForce(company){
  const age = (company.patternAge || 0) / Math.max(1, company.patternLength || 8);
  const primary = triangleWave(age + (company.patternPhaseSeed || 0) * 0.02);
  const micro = (((company.patternAge || 0) % 2) === 0 ? 1 : -1) * 0.28;
  let shape = primary * (company.patternCompression || 0.9) + micro;
  let bias = company.patternBias || 0;
  switch(company.patternMode){
    case 'triangle-up':
      shape *= (1 - age * 0.48);
      bias += 0.16 + age * 0.38;
      break;
    case 'triangle-down':
      shape *= (1 - age * 0.48);
      bias -= 0.16 + age * 0.38;
      break;
    case 'flag-up':
      shape *= 0.68;
      bias += 0.26;
      break;
    case 'flag-down':
      shape *= 0.68;
      bias -= 0.26;
      break;
    case 'bottoming':
      bias += -0.12 + age * 0.82;
      shape *= 0.88;
      break;
    case 'topping':
      bias += 0.18 - age * 0.95;
      shape *= 0.88;
      break;
    default:
      bias *= 0.42;
      break;
  }
  return {shape, bias, age};
}

function getMilitaryCatalog(){
  return {
    infantry:{id:'infantry', name:'歩兵旅団', icon:'🪖', cost:1800, pol:8, weeks:12, power:4.8, note:'数を揃えやすい基礎戦力。防衛力の底上げに向く。'},
    tanks:{id:'tanks', name:'戦車大隊', icon:'🛡', cost:4600, pol:14, weeks:20, power:11.5, note:'重装甲。陸戦と戦争の前線押し上げに強い。'},
    fighters:{id:'fighters', name:'戦闘機隊', icon:'✈', cost:8200, pol:18, weeks:28, power:16.5, note:'航空優勢を確保し、戦争時の押し込みを強める。'},
    ships:{id:'ships', name:'艦艇隊', icon:'🚢', cost:9600, pol:18, weeks:32, power:18.5, note:'海上戦力。物流線と抑止力にも効く。'},
    drones:{id:'drones', name:'無人機群', icon:'🛸', cost:3800, pol:12, weeks:18, power:8.6, note:'比較的早く揃う高効率戦力。サイバーとも相性が良い。'},
    spies:{id:'spies', name:'諜報班', icon:'🕵', cost:2400, pol:14, weeks:16, power:1.1, note:'潜入して敵国の軍事・経済情報を持ち帰る特殊戦力。諜報成功で情報奪取と資金回収が狙える。'},
  };
}

const MILITARY_ART = {
  infantry:{
    labels:['現代歩兵','強化歩兵','次世代歩兵'],
    paths:[
      'assets/military-art/Modern%20Infantry%20Wide%204.png',
      'assets/military-art/Future%20Infantry%202126%20-%203.png',
      'assets/military-art/Bio%20Warrior%20Armed%203.png',
    ],
  },
  tanks:{
    labels:['現代戦車','発展戦車','未来戦車'],
    paths:[
      'assets/military-art/Modern%20Tank%203.png',
      'assets/military-art/Future%20Tank%202126%20-%202.png',
      'assets/military-art/Future%20Tank%202126%20-%203.png',
    ],
  },
  fighters:{
    labels:['現代戦闘機','高機動戦闘機','次世代戦闘機'],
    paths:[
      'assets/military-art/fighter-modern.png',
      'assets/military-art/fighter-advanced.png',
      'assets/military-art/fighter-future.png',
    ],
  },
  ships:{
    labels:['現代戦艦','未来戦艦','次世代戦艦'],
    paths:[
      'assets/military-art/Modern%20Battleship%201.png',
      'assets/military-art/Future%20Battleship%201.png',
      'assets/military-art/Year%203026%20Battleship%204.png',
    ],
  },
  drones:{
    labels:['現代無人機','攻撃無人機','次世代無人機'],
    paths:[
      'assets/military-art/drone-modern.png',
      'assets/military-art/drone-advanced.png',
      'assets/military-art/drone-future.png',
    ],
  },
  spies:{
    labels:['現代諜報員','高度潜入班','戦略諜報網'],
    paths:[
      'assets/military-art/spy-modern.png',
      'assets/military-art/spy-advanced.png',
      'assets/military-art/spy-future.png',
    ],
  },
};

function getMilitarySuppliers(unitType){
  const trackMap = {
    infantry:['infantry'],
    tanks:['armor'],
    fighters:['air'],
    ships:['naval'],
    drones:['drone'],
    spies:['drone'],
  };
  const sectorMap = {
    infantry:['defense','tech'],
    tanks:['defense','auto','construction'],
    fighters:['defense','space','tech'],
    ships:['shipping','defense','construction'],
    drones:['tech','defense','space'],
    spies:['tech','telecom','defense','finance'],
  };
  return companies.filter(company => {
    if((sectorMap[unitType] || []).includes(company.sector)) return true;
    return (trackMap[unitType] || []).includes(company.selectedTechTrack);
  });
}

function getMilitarySupplierMetrics(unitType){
  const suppliers = getMilitarySuppliers(unitType);
  if(!suppliers.length){
    return {
      suppliers:[],
      avgDepth:20,
      avgFavor:50,
      avgOwnership:0,
      trackRatio:0,
    };
  }
  const hitCount = suppliers.reduce((count, company) => {
    const trackHit =
      (unitType === 'infantry' && company.selectedTechTrack === 'infantry') ||
      (unitType === 'tanks' && company.selectedTechTrack === 'armor') ||
      (unitType === 'fighters' && company.selectedTechTrack === 'air') ||
      (unitType === 'ships' && company.selectedTechTrack === 'naval') ||
      (unitType === 'drones' && company.selectedTechTrack === 'drone') ||
      (unitType === 'spies' && ['drone','air'].includes(company.selectedTechTrack));
    return count + (trackHit ? 1 : 0);
  }, 0);
  return {
    suppliers,
    avgDepth:suppliers.reduce((sum, company) => sum + (company.techDepth || 20), 0) / suppliers.length,
    avgFavor:suppliers.reduce((sum, company) => sum + (company.favorability || 50), 0) / suppliers.length,
    avgOwnership:suppliers.reduce((sum, company) => sum + clamp((S.ownedStocks[company.name] || 0) / Math.max(1, company.sharesOutstanding), 0, 1), 0) / suppliers.length,
    trackRatio:hitCount / suppliers.length,
  };
}

function getMilitaryCompanyBoost(unitType){
  const metrics = getMilitarySupplierMetrics(unitType);
  if(!metrics.suppliers.length) return 0;
  const boost =
    metrics.avgOwnership * 0.32
    + Math.max(0, metrics.avgFavor - 50) * 0.0022
    + Math.max(0, metrics.avgDepth - 28) * 0.0021
    + metrics.trackRatio * 0.24;
  return clamp(boost, 0, 1.35);
}

function getMilitaryQualityProfile(unitType, boostValue=null){
  const art = MILITARY_ART[unitType];
  const boost = boostValue == null ? getMilitaryCompanyBoost(unitType) : boostValue;
  const metrics = getMilitarySupplierMetrics(unitType);
  const techLift = clamp((S.techLevel - 12) / 180, 0, 0.17);
  const doctrineLift = clamp((S.defPower || 0) / 26000, 0, 0.1);
  const spaceLift = clamp((S.space?.level || 0) / 90, 0, 0.05);
  const depthLift = clamp((metrics.avgDepth - 28) / 520, 0, 0.28);
  const favorLift = clamp((metrics.avgFavor - 50) / 260, 0, 0.08);
  const ownershipLift = clamp(metrics.avgOwnership * 0.08, 0, 0.08);
  const trackLift = clamp(metrics.trackRatio * 0.08, 0, 0.08);
  const qualityMultiplier = clamp(
    1
      + boost * 0.075
      + depthLift * 0.36
      + techLift * 0.24
      + doctrineLift * 0.18
      + favorLift * 0.14
      + ownershipLift * 0.2
      + trackLift * 0.16
      + spaceLift * 0.08,
    1,
    1.78
  );
  const maturityScore =
    1
    + boost * 0.08
    + depthLift
    + techLift
    + doctrineLift
    + favorLift
    + ownershipLift
    + trackLift
    + spaceLift;
  let tier = 0;
  if(art){
    if(maturityScore >= 1.56 && metrics.avgDepth >= 120 && S.techLevel >= 28 && metrics.trackRatio >= 0.16) tier = 1;
    if(maturityScore >= 1.98 && metrics.avgDepth >= 240 && S.techLevel >= 42 && (S.defPower || 0) >= 220 && metrics.trackRatio >= 0.3) tier = 2;
  }
  return {
    tier,
    label:art ? art.labels[tier] : '',
    nextLabel:art ? (tier < art.labels.length - 1 ? art.labels[tier + 1] : '最終世代') : '',
    image:art ? art.paths[tier] : '',
    labels:art ? art.labels : [],
    maturity:Math.round(clamp((maturityScore - 1) / 1.02, 0, 1.2) * 100),
    qualityMultiplier,
    quality:qualityMultiplier,
    boost,
    suppliers:metrics.suppliers,
    avgDepth:metrics.avgDepth,
    weeksMultiplier:clamp(1 - boost * 0.045 - depthLift * 0.18 - trackLift * 0.08, 0.84, 1.04),
    costMultiplier:clamp(1 - boost * 0.04 - depthLift * 0.13 - ownershipLift * 0.08, 0.86, 1.05),
    powerMultiplier:1 + boost * 0.16 + depthLift * 0.24 + techLift * 0.2 + doctrineLift * 0.16 + trackLift * 0.1,
  };
}

function getMilitaryQualityMultiplier(unitType, boostValue=null){
  return getMilitaryQualityProfile(unitType, boostValue).qualityMultiplier;
}

function getMilitaryVisualProfile(unitType, boostValue=null){
  const profile = getMilitaryQualityProfile(unitType, boostValue);
  return profile.image ? profile : null;
}

function getMilitaryPowerContribution(){
  const inv = S.militaryInventory || {};
  return (inv.infantry || 0) * 1.6
    + (inv.tanks || 0) * 5.2
    + (inv.fighters || 0) * 7.4
    + (inv.ships || 0) * 8.6
    + (inv.drones || 0) * 3.8
    + (inv.spies || 0) * 0.45;
}

function getMilitaryIndustrySectors(unitType){
  const sectorsSet = new Set(['defense','tech']);
  if(unitType === 'infantry') sectorsSet.add('telecom');
  if(unitType === 'tanks') { sectorsSet.add('auto'); sectorsSet.add('construction'); }
  if(unitType === 'fighters') { sectorsSet.add('space'); sectorsSet.add('telecom'); }
  if(unitType === 'ships') { sectorsSet.add('shipping'); sectorsSet.add('construction'); }
  if(unitType === 'drones') { sectorsSet.add('telecom'); sectorsSet.add('space'); }
  if(unitType === 'spies') { sectorsSet.add('telecom'); sectorsSet.add('finance'); }
  return sectorsSet;
}

function applyMilitaryIndustryWave(unitType, qty=1, phase='start'){
  const sectorsSet = getMilitaryIndustrySectors(unitType);
  const multiplier = Math.max(1, qty);
  companies.filter(company => sectorsSet.has(company.sector)).forEach(company => {
    const startPhase = phase === 'start';
    company.price = Math.max(100, Math.round(company.price * rand(startPhase ? 1.006 : 1.012, startPhase ? 1.026 : 1.052)));
    company.revenue *= rand(startPhase ? 1.004 : 1.012, startPhase ? 1.02 : 1.038) * (1 + Math.min(0.05, multiplier * 0.002));
    company.favorability = clamp((company.favorability || 50) + (startPhase ? 0.28 : 0.58) + multiplier * 0.02, 0, 100);
    company.techDepth += startPhase ? 0.04 : 0.1;
  });
}

window.buildMilitaryUnit = function(unitType, qty=1){
  const unit = getMilitaryCatalog()[unitType];
  if(!unit) return;
  qty = Math.max(1, Math.round(qty || 1));
  const boost = getMilitaryCompanyBoost(unitType);
  const qualityProfile = getMilitaryQualityProfile(unitType, boost);
  const qualityMultiplier = qualityProfile.qualityMultiplier;
  const weeks = Math.max(8, Math.round(unit.weeks * qualityProfile.weeksMultiplier));
  const cost = Math.round(unit.cost * qualityProfile.costMultiplier * qty);
  if(S.money < cost){ addEvent('❌ 資金不足', 'bad'); return; }
  if(S.polCap < unit.pol * qty){ addEvent('❌ 政治資本不足', 'bad'); return; }
  S.money -= cost;
  S.polCap -= unit.pol * qty;
  S.militaryProjects.push({
    id:`mil-${Date.now()}-${Math.random().toString(36).slice(2,6)}`,
    type:unitType,
    qty,
    totalWeeks:weeks,
    weeksLeft:weeks,
    powerGain:unit.power * qualityProfile.powerMultiplier * qty,
    quality:qualityMultiplier,
    suppliers:getMilitarySuppliers(unitType).slice(0, 3).map(company => company.name),
  });
  getMilitarySuppliers(unitType).forEach(company => {
    company.price = Math.round(company.price * rand(1.008, 1.03));
    company.revenue *= rand(1.004, 1.018);
    company.favorability = Math.min(100, (company.favorability || 50) + 0.8);
  });
  applyMilitaryIndustryWave(unitType, qty, 'start');
  addEvent(`${unit.icon} ${unit.name} x${qty} の生産を開始。完成まで ${weeks}週`, 'good');
  renderPanel();
  updateUI();
};

function progressMilitaryProjects(){
  if(!S.militaryProjects.length) return;
  const completed = [];
  S.militaryProjects.forEach(project => {
    project.weeksLeft--;
    if(project.weeksLeft <= 0) completed.push(project);
  });
  S.militaryProjects = S.militaryProjects.filter(project => project.weeksLeft > 0);
  completed.forEach(project => {
    const unit = getMilitaryCatalog()[project.type];
    if(!unit) return;
    S.militaryInventory[project.type] = (S.militaryInventory[project.type] || 0) + (project.qty || 1);
    S.defPower += project.powerGain;
    project.suppliers.forEach(name => {
      const company = getCompanyByName(name);
      if(!company) return;
      company.techDepth += 0.6;
      company.favorability = Math.min(100, (company.favorability || 50) + 1.4);
      company.price = Math.round(company.price * rand(1.01, 1.04));
      company.revenue *= rand(1.01, 1.03);
    });
    applyMilitaryIndustryWave(project.type, project.qty || 1, 'complete');
    addEvent(`${unit.icon} ${unit.name} x${project.qty || 1} が配備完了。防衛力 +${project.powerGain.toFixed(1)}`, 'good');
  });
}

const spaceBodies = [
  {id:'moon',name:'月',color:'#cfd8dc',difficulty:28, travelWeeks:10, reward:18, materials:['helium3','regolith_alloy'], desc:'最初の恒常拠点候補。ヘリウム3と建材の採掘に向く。'},
  {id:'mars',name:'火星',color:'#ff8a65',difficulty:46, travelWeeks:18, reward:34, materials:['regolith_alloy','cryo_water'], desc:'長距離だが産出が大きく、宇宙工業化への橋頭堡になる。'},
  {id:'asteroid',name:'小惑星帯',color:'#b39ddb',difficulty:58, travelWeeks:22, reward:48, materials:['iridium_x','regolith_alloy'], desc:'高難度だが一発の資源価値が大きい。宇宙株を強く押し上げる。'},
  {id:'europa',name:'エウロパ',color:'#80deea',difficulty:64, travelWeeks:26, reward:56, materials:['exo_biomass','cryo_water'], desc:'医療・生命科学向けの希少試料が期待できる。'},
];

function createPlanetResourceState(){
  const state = {};
  spaceBodies.forEach(body => {
    state[body.id] = {
      total:body.materials.reduce((acc, materialId) => {
        acc[materialId] = Math.round(body.reward * rand(3500, 6200));
        return acc;
      }, {}),
      remaining:{},
      developed:false,
    };
    Object.entries(state[body.id].total).forEach(([materialId, total]) => {
      state[body.id].remaining[materialId] = total;
    });
  });
  return state;
}

if(!S.space) S.space = createSpaceState();
if(!S.nuclear) S.nuclear = createNuclearState();
if(!S.space.planets || !Object.keys(S.space.planets).length) S.space.planets = createPlanetResourceState();
ensureStateShape();

const publicWorksCatalog = {
  thermal:{id:'thermal',name:'火力発電所',icon:'🏭',cost:26000,pol:20,output:18,energyDemand:0,buildWeeks:16,upkeep:420,allowed:'land',desc:'安定した出力を持つ大型発電所。稼働率を上げると燃料費も増える。'},
  hydro:{id:'hydro',name:'ダム',icon:'🏞',cost:32000,pol:22,output:24,energyDemand:0,buildWeeks:22,upkeep:290,allowed:'river',desc:'川沿い限定。安定した電力と治水を提供し、建設会社に大型受注を生む。'},
  solar:{id:'solar',name:'太陽光施設',icon:'☀',cost:14000,pol:12,output:9,energyDemand:0,buildWeeks:10,upkeep:150,allowed:'land',desc:'建設しやすい再エネ設備。気象の影響はあるが固定費は軽い。'},
  hospital:{id:'hospital',name:'公立病院',icon:'🏥',cost:18500,pol:14,output:0,energyDemand:4,buildWeeks:14,upkeep:280,allowed:'land',desc:'支持率と安定度に寄与し、災害時の復旧力も高める。'},
  industrial:{id:'industrial',name:'工業団地',icon:'🏗',cost:22500,pol:16,output:0,energyDemand:12,buildWeeks:18,upkeep:470,throughput:14,allowed:'land',desc:'製造業の売上と雇用を押し上げる公共投資。稼働率とエネルギー需要が大きい。'},
};

const transportNetworkCatalog = {
  rail:{id:'rail',name:'高速鉄道',icon:'🚆',cost:17500,pol:14,buildWeeks:18,upkeep:94,productivity:5.8,appeal:7.5,migration:1.15,companyBoost:0.02,desc:'県と県を結ぶ主力交通網。人口流動、生産性、企業売上に広く効く。'},
  expressway:{id:'expressway',name:'高速道路',icon:'🛣',cost:14500,pol:12,buildWeeks:15,upkeep:74,productivity:4.2,appeal:5.2,migration:0.92,companyBoost:0.016,desc:'物流と自動車輸送を強化し、広域の経済循環を改善する。'},
  ferry:{id:'ferry',name:'航路連絡',icon:'⛴',cost:12200,pol:11,buildWeeks:13,upkeep:56,productivity:3.4,appeal:4.1,migration:0.68,companyBoost:0.013,desc:'離れた沿岸県を結ぶ海上交通。島しょ部や港湾経済に効く。'},
  airlink:{id:'airlink',name:'航空回廊',icon:'✈',cost:21000,pol:16,buildWeeks:16,upkeep:104,productivity:4.9,appeal:8.4,migration:0.84,companyBoost:0.021,desc:'遠距離移動と高付加価値物流を改善し、魅力度を大きく押し上げる。'},
};

const coastalPrefectureIds = new Set([
  'hokkaido','aomori','iwate','miyagi','akita','yamagata','fukushima',
  'ibaraki','chiba','tokyo','kanagawa','niigata','toyama','ishikawa','fukui',
  'shizuoka','aichi','mie','kyoto','osaka','hyogo','wakayama',
  'okayama','hiroshima','yamaguchi','tokushima','kagawa','ehime','kochi',
  'fukuoka','saga','nagasaki','kumamoto','oita','miyazaki','kagoshima','okinawa'
]);

const riverPolylines = [
  [[0.63,0.37],[0.60,0.43],[0.57,0.49]],
  [[0.52,0.48],[0.48,0.55],[0.44,0.61]],
  [[0.34,0.57],[0.30,0.63],[0.26,0.67]],
  [[0.72,0.25],[0.71,0.32],[0.69,0.40]],
];

const productCatalog = {
  auto:{name:'自動車製品',price:500,resource:'iron'},
  tech:{name:'技術製品',price:800,resource:'silicon'},
  space_goods:{name:'宇宙関連製品',price:1300,resource:'titanium'},
  medical_goods:{name:'医療製品',price:1450,resource:'hydrogen',requires:['antiviral_platform']},
  advanced_materials:{name:'先端素材製品',price:1700,resource:'neo_carbon',requires:['orbital_fabrication']},
  energy_cells:{name:'次世代電池',price:1550,resource:'helium3',requires:['fusion_grid']},
  quantum_devices:{name:'量子通信機器',price:1820,resource:'iridium_x',requires:['quantum_network']},
  biotech_goods:{name:'バイオ素材製品',price:1680,resource:'exo_biomass',requires:['europa_biotech']},
};

const researchCatalog = [
  {id:'antiviral_platform',name:'抗ウイルス創薬',company:'武野薬品工業',cost:12000,pol:28,weeks:18,minTech:24,minStock:1.02,minRevenueGrowth:1.02,desc:'新型感染症対策の創薬基盤を整備し、医療製品を輸出できるようにする。',unlock:'medical_goods'},
  {id:'orbital_fabrication',name:'軌道素材製造',company:'オービタル重工',cost:15000,pol:34,weeks:22,minTech:28,minStock:1.05,minRevenueGrowth:1.03,requiresMaterial:'regolith_alloy',desc:'宇宙由来素材を使った高収益な先端素材ラインを解放する。',unlock:'advanced_materials'},
  {id:'fusion_grid',name:'核融合電力網',company:'東都電力HD',cost:17000,pol:30,weeks:24,minTech:30,minStock:1.01,minRevenueGrowth:1.02,requiresMaterial:'helium3',desc:'ヘリウム3 を利用した次世代電力供給網。エネルギーと円相場を押し上げる。',unlock:'energy_cells'},
  {id:'quantum_network',name:'量子通信網',company:'NTテレコム',cost:14000,pol:30,weeks:20,minTech:32,minStock:1.04,minRevenueGrowth:1.03,requiresMaterial:'iridium_x',desc:'超高信頼の量子通信を展開し、技術製品の付加価値を引き上げる。',unlock:'quantum_devices'},
  {id:'europa_biotech',name:'宇宙生命科学',company:'日本医療HD',cost:15500,pol:32,weeks:22,minTech:29,minStock:1.02,minRevenueGrowth:1.03,requiresMaterial:'exo_biomass',desc:'宇宙由来試料を活用した医療・バイオ産業を拡張する。',unlock:'biotech_goods'},
  {id:'smart_mobility',name:'次世代モビリティ',company:'トヨダ自動車',cost:13500,pol:24,weeks:18,minTech:26,minStock:1.03,minRevenueGrowth:1.03,requiresMaterial:'neo_carbon',desc:'新素材を使った高収益モビリティ製品群を創出し、自動車産業の収益体質を改善する。'},
  {id:'relief_biologics',name:'緊急医療救援網',company:'武野薬品工業',cost:14800,pol:26,weeks:20,minTech:31,minStock:1.04,minRevenueGrowth:1.03,desc:'医療救援用の迅速生物製剤と広域支援ネットワークを整え、災害タブで善性イベントを使えるようにする。'},
  {id:'climate_shield',name:'気象制御シールド',company:'東都電力HD',cost:16800,pol:30,weeks:22,minTech:33,minStock:1.05,minRevenueGrowth:1.03,desc:'送電保護と局地気象制御で、災害軽減と送電網の緊急復旧が可能になる。'},
  {id:'urban_resilience',name:'都市防災デジタルツイン',company:'NTテレコム',cost:15200,pol:28,weeks:21,minTech:34,minStock:1.03,minRevenueGrowth:1.03,desc:'都市防災の予測と即応を高度化し、災害タブに復旧系オプションを追加する。'},
];

const pathogenCatalog = [
  {id:'amber_fever',name:'アンバー熱',company:'武野薬品工業',cost:16000,pol:38,weeks:16,minTech:30,minStock:1.05,minRevenueGrowth:1.04,desc:'高感染性の呼吸器病原体。散布すると強いパンデミックを引き起こす。'},
  {id:'velvet_strain',name:'ベルベット株',company:'武野薬品工業',cost:22000,pol:46,weeks:20,minTech:36,minStock:1.08,minRevenueGrowth:1.05,requiresMaterial:'exo_biomass',desc:'高難度の人工変異株。成功すれば世界経済に大打撃を与える。'},
];

tradePartners.forEach(partner => {
  ['medical_goods','advanced_materials','energy_cells','quantum_devices','biotech_goods','space_goods','tech','auto'].forEach(product => {
    if(!partner.buys.includes(product) && Math.random() < 0.68) partner.buys.push(product);
  });
});

function hasCompletedResearch(id){ return S.completedResearch.includes(id); }
function getProductUnlockProject(productId){
  return researchCatalog.find(project => project.unlock === productId) || null;
}
function getProductIndustrializationDetail(productId){
  if(!S.productIndustrializationDetail || typeof S.productIndustrializationDetail !== 'object') S.productIndustrializationDetail = {};
  const existing = S.productIndustrializationDetail[productId];
  if(existing && typeof existing === 'object'){
    return {
      certification:clamp(existing.certification || 0, 0, 1),
      line:clamp(existing.line || 0, 0, 1),
      supply:clamp(existing.supply || 0, 0, 1),
      channel:clamp(existing.channel || 0, 0, 1),
    };
  }
  const overall = clamp(S.productIndustrialization?.[productId] || 0, 0, 1);
  const seeded = overall > 0 ? overall : 0;
  const detail = {certification:seeded, line:seeded, supply:seeded, channel:seeded};
  S.productIndustrializationDetail[productId] = detail;
  return detail;
}
function getProductCommercializationStatus(productId){
  const product = productCatalog[productId];
  if(!product) return null;
  if(!product.requires?.length){
    return {
      product,
      overall:1,
      detail:{certification:1,line:1,supply:1,channel:1},
      missing:[],
      company:null,
      prefectureStat:null,
      transportLinks:0,
      industrialWorks:0,
    };
  }
  const detail = getProductIndustrializationDetail(productId);
  const project = getProductUnlockProject(productId);
  const company = project ? getCompanyByName(project.company) : null;
  const prefectureStat = company?.prefecture ? getPrefectureStat(company.prefecture) : null;
  const transportLinks = company?.prefecture ? getPrefectureTransportRoutes(company.prefecture, true).length : 0;
  const industrialWorks = (S.publicWorks || []).filter(work => work.status === 'active' && (work.type === 'industrial' || work.type === 'hospital') && (!company || work.region === company.region)).length;
  const resource = resources[product.resource];
  const resourceRatio = resource?.max ? clamp(resource.amount / Math.max(1, resource.max), 0, 1.4) : (resource?.amount > 0 ? 0.35 : 0);
  const energyMargin = (S.energy?.production || 0) - (S.energy?.demand || 0);
  const demandPartners = tradePartners.filter(partner => partner.buys?.includes(productId)).length;
  const missing = [];
  if((company?.favorability || 0) < 62) missing.push('企業好感度');
  if((company?.techDepth || 0) < 32) missing.push('技術深度');
  if((prefectureStat?.productivity || 0) < 6) missing.push('県生産性');
  if(industrialWorks < 1) missing.push('量産拠点');
  if(resourceRatio < 0.22) missing.push('素材供給');
  if(energyMargin < 4) missing.push('電力余裕');
  if(transportLinks < 1) missing.push('物流網');
  if(demandPartners < 2) missing.push('販路');
  return {
    product,
    project,
    company,
    prefectureStat,
    transportLinks,
    industrialWorks,
    demandPartners,
    resourceRatio,
    energyMargin,
    overall:clamp((detail.certification + detail.line + detail.supply + detail.channel) / 4, 0, 1),
    detail,
    missing,
  };
}
function getProductIndustrialization(productId){
  const product = productCatalog[productId];
  if(!product) return 0;
  if(!product.requires || !product.requires.length) return 1;
  const status = getProductCommercializationStatus(productId);
  const fallback = typeof S.productIndustrialization?.[productId] === 'number' ? S.productIndustrialization[productId] : 0;
  return clamp(status ? status.overall : fallback, 0, 1);
}
function getProductIndustrializationLabel(productId){
  const status = getProductCommercializationStatus(productId);
  const progress = clamp(status?.overall ?? getProductIndustrialization(productId), 0, 1);
  if(progress >= 1) return '量産化完了';
  if(progress <= 0) return '量産化準備中';
  if(status?.missing?.length){
    return `量産化 ${Math.round(progress * 100)}% / 不足: ${status.missing.slice(0, 2).join('・')}`;
  }
  return `量産化 ${Math.round(progress * 100)}%`;
}
function hasUnlockedProduct(id){
  const product = productCatalog[id];
  if(!product) return false;
  if(!product.requires || !product.requires.length) return true;
  return product.requires.every(hasCompletedResearch) && getProductIndustrialization(id) >= 1;
}
function getCompanyByName(name){ return companies.find(company => company.name === name); }
function hasFullControl(name){
  const company = getCompanyByName(name);
  if(!company) return false;
  return (S.ownedStocks[name] || 0) >= company.sharesOutstanding;
}
function getHistoricReference(series, weeks){
  if(!series?.length) return 0;
  return series[Math.max(0, series.length - weeks)] || series[0] || 0;
}
function getResearchCompanyStatus(company){
  if(!company) return {stockRatio:0, revenueRatio:0};
  const revenueBase = Math.max(1, getHistoricReference(company.revenueHist, 52));
  return {
    stockRatio:company.base ? company.price / company.base : 1,
    revenueRatio:company.revenue / revenueBase,
  };
}
function canStartResearchProject(project){
  const company = getCompanyByName(project.company);
  if(!company) return {ok:false, reason:'企業が存在しません'};
  const status = getResearchCompanyStatus(company);
  if(!hasFullControl(project.company)) return {ok:false, reason:'対象企業を100%保有していません'};
  if(S.money < project.cost) return {ok:false, reason:'資金不足'};
  if(S.polCap < project.pol) return {ok:false, reason:'政治資本不足'};
  if(S.techLevel < project.minTech) return {ok:false, reason:`技術力 ${project.minTech} 以上が必要`};
  if((company.favorability || 0) < 55) return {ok:false, reason:'対象企業の国への好感度が55以上必要です'};
  if(status.stockRatio < project.minStock) return {ok:false, reason:'株価評価が不足しています'};
  if(status.revenueRatio < project.minRevenueGrowth) return {ok:false, reason:'売上成長が不足しています'};
  if(project.requiresMaterial && (!resources[project.requiresMaterial] || resources[project.requiresMaterial].amount <= 0)) return {ok:false, reason:`${resources[project.requiresMaterial]?.name || project.requiresMaterial} が必要`};
  return {ok:true, reason:'開始できます', company, status};
}
function unlockProduct(id){
  if(!id || S.unlockedProducts.includes(id)) return;
  S.unlockedProducts.push(id);
  if(productCatalog[id]?.requires?.length){
    S.productIndustrialization[id] = 0;
    S.productIndustrializationDetail[id] = {certification:0, line:0, supply:0, channel:0};
    delete S.productIndustrializationAnnounced[id];
  }
}

function progressProductIndustrialization(){
  Object.entries(productCatalog).forEach(([productId, product]) => {
    if(!product.requires?.length) return;
    if(!product.requires.every(hasCompletedResearch)) return;
    const status = getProductCommercializationStatus(productId);
    const detail = getProductIndustrializationDetail(productId);
    const favorability = status.company?.favorability || 50;
    const techDepth = status.company?.techDepth || 20;
    const prefectureProductivity = status.prefectureStat?.productivity || 0;
    const certificationGain = (favorability >= 58 && techDepth >= 26)
      ? clamp(0.006 + S.techLevel * 0.00016 + Math.max(0, techDepth - 22) * 0.00022 + Math.max(0, favorability - 55) * 0.00012, 0.004, 0.028)
      : -0.0024;
    const lineGain = (status.industrialWorks >= 1 && prefectureProductivity >= 0)
      ? clamp(0.004 + status.industrialWorks * 0.0036 + Math.max(0, prefectureProductivity) * 0.00035 + S.infraLevel * 0.00018, 0.003, 0.026)
      : -0.0028;
    const supplyGain = (status.resourceRatio >= 0.12 && status.energyMargin >= 0)
      ? clamp(0.004 + status.resourceRatio * 0.016 + Math.max(0, status.energyMargin) * 0.00016, 0.003, 0.024)
      : -0.0032;
    const channelGain = (status.transportLinks >= 1 && status.demandPartners >= 1)
      ? clamp(0.0035 + status.transportLinks * 0.0028 + status.demandPartners * 0.0022 + Math.max(0, status.company?.favorability - 50) * 0.00009, 0.003, 0.024)
      : -0.0025;
    detail.certification = clamp(detail.certification + certificationGain, 0, 1);
    detail.line = clamp(detail.line + lineGain, 0, 1);
    detail.supply = clamp(detail.supply + supplyGain, 0, 1);
    detail.channel = clamp(detail.channel + channelGain, 0, 1);
    S.productIndustrializationDetail[productId] = detail;
    const next = clamp((detail.certification + detail.line + detail.supply + detail.channel) / 4, 0, 1);
    S.productIndustrialization[productId] = next;
    if(next >= 1 && !S.productIndustrializationAnnounced[productId]){
      S.productIndustrializationAnnounced[productId] = true;
      addEvent(`🏭 ${product.name} の量産化が完了。輸出に使えるようになった`, 'good');
    }
  });
}
function resolveResearchProject(project){
  if(project.kind === 'pathogen'){
    if(!S.bioWeaponUnlocks.includes(project.id)) S.bioWeaponUnlocks.push(project.id);
    S.techLevel += 1.2;
    addEvent(`🧪 特殊研究「${project.name}」完了。病原体を散布できるようになった`, 'bad');
    return;
  }
  if(!S.completedResearch.includes(project.id)) S.completedResearch.push(project.id);
  unlockProduct(project.unlock);
  if(project.id === 'antiviral_platform'){
    companies.filter(c => ['pharma','hospital'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(1.03, 1.08)));
    S.techLevel += 1.4;
    S.approval = Math.min(100, S.approval + 1.2);
  } else if(project.id === 'orbital_fabrication'){
    companies.filter(c => ['space','construction','mining'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(1.03, 1.09)));
    S.techLevel += 1.8;
  } else if(project.id === 'fusion_grid'){
    S.energyBonus += 3;
    S.yenValue = Math.min(160, S.yenValue + 2.2);
    companies.filter(c => ['energy','tech'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(1.03, 1.08)));
  } else if(project.id === 'quantum_network'){
    S.cyberPower += 4;
    S.techLevel += 2.1;
    companies.filter(c => ['telecom','tech'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(1.04, 1.1)));
  } else if(project.id === 'europa_biotech'){
    companies.filter(c => ['hospital','pharma','food'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(1.03, 1.08)));
    S.techLevel += 1.5;
  } else if(project.id === 'smart_mobility'){
    companies.filter(c => ['auto','retail','tech'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(1.02, 1.07)));
    S.techLevel += 1.2;
  } else if(project.id === 'relief_biologics'){
    companies.filter(c => ['hospital','pharma','food'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(1.03, 1.08)));
    S.approval = Math.min(100, S.approval + 1.4);
    S.stability = Math.min(100, S.stability + 1.2);
    S.techLevel += 1.3;
  } else if(project.id === 'climate_shield'){
    companies.filter(c => ['energy','construction','telecom'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(1.03, 1.08)));
    S.energyBonus += 1.5;
    S.techLevel += 1.4;
  } else if(project.id === 'urban_resilience'){
    companies.filter(c => ['telecom','construction','hospital'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(1.03, 1.08)));
    S.stability = Math.min(100, S.stability + 1.5);
    S.techLevel += 1.5;
  }
  addEvent(`🧪 研究「${project.name}」が完了。新しい産業連鎖が解放された`, 'good');
}
function triggerBioOutbreak(pathogen){
  const duration = 104;
  S.activeMajorEvent = {id:'bio_outbreak', name:`${pathogen.name}拡散`, weeksLeft:duration, duration, pathogenId:pathogen.id, sourceCompany:pathogen.company};
  S.majorEventCooldown = 156;
  companies.filter(c => ['shipping','retail','auto','food','finance'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(0.56, 0.8)));
  companies.filter(c => ['hospital','pharma'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(1.08, 1.24)));
  S.gdp *= 0.978;
  S.approval = Math.max(0, S.approval - 5);
  S.stability = Math.max(0, S.stability - 7);
  showMajorEventBanner(`重大イベント: ${pathogen.name} が拡散`);
  addEvent(`☣ ${pathogen.company} 由来の ${pathogen.name} を世界に拡散。医療と物流が大混乱`, 'bad');
}

// ============================================================
// MAP
// ============================================================
const canvas = document.getElementById('mapCanvas');
const ctx = canvas.getContext('2d');
let mapW, mapH;

const referenceMaskCache = {};
const referenceSyncState = {japan:false, world:false};

function createLocalMapArt(kind, filePath){
  const image = new Image();
  image.decoding = 'async';
  image.loading = 'eager';
  image.src = encodeURI(`file:///${filePath.replace(/\\/g,'/')}`);
  image.onload = () => {
    referenceMaskCache[kind] = null;
    referenceSyncState[kind] = false;
    if(typeof drawMap === 'function'){
      try{ drawMap(); }catch(error){ reportRuntimeError('mapArt:onload', error); }
    }
  };
  return image;
}

const mapArt = {
  japan:createLocalMapArt('japan', 'C:\\Users\\Admin\\Downloads\\日本マップ１.png'),
  japanAlt:createLocalMapArt('japanAlt', 'C:\\Users\\Admin\\Downloads\\日本マップ.png'),
  world:createLocalMapArt('world', 'C:\\Users\\Admin\\Downloads\\clker-free-vector-images-map-305667_1920.png'),
  worldAlt:createLocalMapArt('worldAlt', 'C:\\Users\\Admin\\Downloads\\世界マップ.webp'),
};

// Japan polygons (more detailed)
const japanPolys = {
  hokkaido: [[.66,.08],[.68,.06],[.72,.05],[.76,.05],[.79,.06],[.81,.08],[.82,.11],[.81,.14],[.79,.17],[.76,.19],[.73,.21],[.70,.20],[.68,.18],[.66,.15],[.65,.12]],
  tohoku: [[.67,.24],[.70,.23],[.73,.24],[.75,.26],[.76,.29],[.76,.32],[.75,.35],[.73,.37],[.71,.38],[.69,.37],[.67,.35],[.66,.32],[.65,.29],[.66,.26]],
  kanto: [[.66,.39],[.69,.38],[.72,.39],[.74,.41],[.74,.44],[.73,.47],[.71,.48],[.68,.48],[.66,.47],[.65,.44],[.65,.41]],
  chubu: [[.55,.42],[.58,.41],[.62,.42],[.65,.44],[.65,.47],[.63,.49],[.60,.50],[.57,.50],[.54,.48],[.53,.45],[.54,.43]],
  kinki: [[.48,.48],[.51,.47],[.54,.48],[.55,.51],[.54,.54],[.52,.55],[.49,.55],[.47,.53],[.46,.50]],
  chugoku: [[.35,.48],[.38,.47],[.42,.48],[.45,.50],[.46,.52],[.44,.54],[.41,.55],[.38,.54],[.35,.52],[.34,.50]],
  shikoku: [[.39,.57],[.42,.56],[.46,.57],[.47,.60],[.46,.63],[.43,.64],[.40,.63],[.38,.60]],
  kyushu: [[.24,.56],[.28,.55],[.32,.56],[.34,.58],[.34,.62],[.32,.66],[.30,.68],[.27,.68],[.24,.66],[.22,.62],[.22,.58]],
  okinawa: [[.13,.82],[.15,.81],[.17,.82],[.18,.84],[.17,.87],[.15,.88],[.13,.87],[.12,.84]],
};

ensureCompanyMapPositions();

function resizeCanvas(){
  const rect = canvas.parentElement.getBoundingClientRect();
  canvas.width = Math.max(2, Math.floor(rect.width * 2));
  canvas.height = Math.max(2, Math.floor(rect.height * 2));
  mapW = canvas.width; mapH = canvas.height;
  if(mapW < 4 || mapH < 4) return;
  positionMapHud();
  drawMap();
}

function getModeImageAspect(mode=S.currentMapMode){
  if(mode === 'space') return mapW / Math.max(1, mapH);
  const image = mode === 'world' ? mapArt.world : mapArt.japan;
  if(image?.complete && image.naturalWidth && image.naturalHeight) return image.naturalWidth / image.naturalHeight;
  return mode === 'world' ? 1.68 : 1;
}

function getModeLogicalBounds(mode=S.currentMapMode){
  return {x:0, y:0, w:1, h:1};
}

function getReferenceMask(kind){
  if(referenceMaskCache[kind] === false) return null;
  if(referenceMaskCache[kind]) return referenceMaskCache[kind];
  const image = mapArt[kind];
  if(!image?.complete || !image.naturalWidth || !image.naturalHeight) return null;
  try{
    const offscreen = document.createElement('canvas');
    offscreen.width = image.naturalWidth;
    offscreen.height = image.naturalHeight;
    const offctx = offscreen.getContext('2d', {willReadFrequently:true});
    offctx.drawImage(image, 0, 0);
    const mask = {
      width:offscreen.width,
      height:offscreen.height,
      data:offctx.getImageData(0, 0, offscreen.width, offscreen.height).data,
    };
    referenceMaskCache[kind] = mask;
    return mask;
  } catch(error){
    referenceMaskCache[kind] = false;
    console.warn(`reference mask disabled for ${kind}:`, error?.message || error);
    return null;
  }
}

function sampleReferenceLand(kind, nx, ny){
  const mask = getReferenceMask(kind);
  if(!mask) return null;
  const logical = getModeLogicalBounds(kind);
  const u = logical.x + nx * logical.w;
  const v = logical.y + ny * logical.h;
  if(u < 0 || u > 1 || v < 0 || v > 1) return 0;
  const px = clamp(Math.round(u * (mask.width - 1)), 0, mask.width - 1);
  const py = clamp(Math.round(v * (mask.height - 1)), 0, mask.height - 1);
  const idx = (py * mask.width + px) * 4;
  return mask.data[idx] + mask.data[idx + 1] + mask.data[idx + 2];
}

function isReferenceLand(kind, nx, ny, threshold=48){
  const score = sampleReferenceLand(kind, nx, ny);
  return score === null ? false : score > threshold;
}

function snapPointToReferenceLand(kind, x, y, fallbackX=x, fallbackY=y, radius=0.03){
  const mask = getReferenceMask(kind);
  if(!mask) return {x, y};
  const startingPoints = [
    {x, y, penalty:0},
    {x:fallbackX, y:fallbackY, penalty:0.003},
  ];
  let best = null;
  startingPoints.forEach(point => {
    const score = sampleReferenceLand(kind, point.x, point.y);
    if(score !== null && score > 48){
      const weighted = score - point.penalty * 1000;
      if(!best || weighted > best.weighted) best = {...point, weighted};
    }
  });
  for(let currentRadius = 0.0015; currentRadius <= radius; currentRadius += 0.0015){
    for(let step = 0; step < 36; step++){
      const angle = Math.PI * 2 * (step / 36);
      [ {x, y, penalty:0}, {x:fallbackX, y:fallbackY, penalty:0.002} ].forEach(origin => {
        const candidate = {
          x:origin.x + Math.cos(angle) * currentRadius,
          y:origin.y + Math.sin(angle) * currentRadius,
        };
        const score = sampleReferenceLand(kind, candidate.x, candidate.y);
        if(score === null || score <= 48) return;
        const dist = Math.hypot(candidate.x - fallbackX, candidate.y - fallbackY);
        const weighted = score - dist * 1600 - origin.penalty * 1000;
        if(!best || weighted > best.weighted){
          best = {...candidate, weighted};
        }
      });
    }
  }
  return best ? {x:best.x, y:best.y} : {x:fallbackX, y:fallbackY};
}

function syncJapanReferenceAnchors(){
  if(referenceSyncState.japan) return;
  applyPrefectureGeoOverrides();
  ensureCompanyMapPositions();
  referenceSyncState.japan = true;
}

function syncWorldReferenceAnchors(){
  if(referenceSyncState.world) return;
  referenceSyncState.world = true;
}

function getMapViewport(mode=S.currentMapMode){
  if(mode === 'space') return {x:0, y:0, w:mapW, h:mapH};
  const padding = mode === 'world' ? 0.008 : 0.016;
  const innerW = mapW * (1 - padding * 2);
  const innerH = mapH * (1 - padding * 2);
  const imageAspect = getModeImageAspect(mode);
  let drawW = innerW;
  let drawH = innerW / imageAspect;
  if(drawH > innerH){
    drawH = innerH;
    drawW = drawH * imageAspect;
  }
  return {
    x:(mapW - drawW) / 2,
    y:(mapH - drawH) / 2,
    w:drawW,
    h:drawH,
  };
}

function worldToCanvas(nx, ny){
  const logical = getModeLogicalBounds(S.currentMapMode);
  const viewport = getMapViewport(S.currentMapMode);
  const ux = logical.x + nx * logical.w;
  const uy = logical.y + ny * logical.h;
  return {
    x:viewport.x + viewport.w * 0.5 + ((ux - 0.5 + S.mapOffsetX) * viewport.w * S.mapZoom),
    y:viewport.y + viewport.h * 0.5 + ((uy - 0.5 + S.mapOffsetY) * viewport.h * S.mapZoom)
  };
}

function canvasToWorld(px, py){
  const logical = getModeLogicalBounds(S.currentMapMode);
  const viewport = getMapViewport(S.currentMapMode);
  const ux = ((px - (viewport.x + viewport.w * 0.5)) / (viewport.w * S.mapZoom)) + 0.5 - S.mapOffsetX;
  const uy = ((py - (viewport.y + viewport.h * 0.5)) / (viewport.h * S.mapZoom)) + 0.5 - S.mapOffsetY;
  return {
    x:(ux - logical.x) / Math.max(1e-9, logical.w),
    y:(uy - logical.y) / Math.max(1e-9, logical.h)
  };
}

function getMapViewKey(mode){
  if(mode === 'space') return 'spaceView';
  if(mode === 'world') return 'worldView';
  return 'japanView';
}

function getDefaultMapView(mode){
  if(mode === 'space') return {offsetX:-0.02, offsetY:-0.03, zoom:1.02};
  if(mode === 'world') return {offsetX:0, offsetY:0, zoom:1.02};
  return {offsetX:-0.018, offsetY:0.01, zoom:1};
}

function getMapBounds(mode){
  if(mode === 'world') return {minX:-0.94, maxX:0.94, minY:-0.86, maxY:0.86, minZoom:0.76, maxZoom:6.8};
  if(mode === 'space') return {minX:-0.32, maxX:0.32, minY:-0.38, maxY:0.38, minZoom:0.76, maxZoom:4.2};
  return {minX:-0.76, maxX:0.76, minY:-0.9, maxY:0.88, minZoom:0.72, maxZoom:9.4};
}

function positionMapHud(){
  const hud = document.getElementById('mapHud');
  if(!hud || !canvas || !mapW || !mapH) return;
  const areaRect = gameArea.getBoundingClientRect();
  const viewport = getMapViewport(S.currentMapMode);
  const scaleX = areaRect.width / Math.max(1, mapW);
  const scaleY = areaRect.height / Math.max(1, mapH);
  const baseLeft = areaRect.left + viewport.x * scaleX + 16;
  const baseTop = areaRect.top + viewport.y * scaleY + 8;
  const offsetX = S.mapHudOffset?.x || 0;
  const offsetY = S.mapHudOffset?.y || 0;
  const width = hud.offsetWidth || 360;
  const height = hud.offsetHeight || 120;
  const nextLeft = clamp(baseLeft + offsetX, 8, Math.max(8, window.innerWidth - width - 8));
  const nextTop = clamp(baseTop + offsetY, 8, Math.max(8, window.innerHeight - height - 8));
  hud.style.left = `${Math.round(nextLeft)}px`;
  hud.style.top = `${Math.round(nextTop)}px`;
}

function clampMapView(mode=S.currentMapMode){
  const bounds = getMapBounds(mode);
  S.mapZoom = clamp(S.mapZoom, bounds.minZoom, bounds.maxZoom);
  S.mapOffsetX = clamp(S.mapOffsetX, bounds.minX, bounds.maxX);
  S.mapOffsetY = clamp(S.mapOffsetY, bounds.minY, bounds.maxY);
}

function storeCurrentView(){
  const key = getMapViewKey(S.currentMapMode);
  S[key] = {offsetX:S.mapOffsetX, offsetY:S.mapOffsetY, zoom:S.mapZoom};
}

function loadViewForMode(mode){
  const view = S[getMapViewKey(mode)] || getDefaultMapView(mode);
  S.mapOffsetX = view.offsetX;
  S.mapOffsetY = view.offsetY;
  S.mapZoom = view.zoom;
  clampMapView(mode);
}

function getWorldCountry(countryId){
  return worldCountries.find(country => country.id === countryId) || worldCountries[0];
}

function getWorldCountryPartner(country){
  if(!country || typeof country.partnerIdx !== 'number') return null;
  return tradePartners[country.partnerIdx] || null;
}

function getWorldCountrySnapshot(country){
  if(!country) country = getWorldCountry('japan');
  if(country.id === 'japan'){
    return {
      relation:100,
      economy:Math.max(55, Math.min(140, Math.round(S.gdp / 60))),
      cyber:S.cyberPower,
      nuke:S.nukePower,
      gdp:S.gdp,
      actualGdp:JAPAN_REFERENCE_GDP_USD,
      growth:1.2,
      inflation:S.inflation,
      unemployment:S.unemployment,
      currencyName:'円',
      currencyCode:'JPY',
      currencyRate:1,
      hostility:0,
      nuclearScars:0,
      label:'本拠地',
    };
  }
  const partner = getWorldCountryPartner(country);
  return {
    relation:partner?.relation ?? 50,
    economy:partner?.economy ?? 40,
    cyber:partner?.cyberPower ?? 20,
    nuke:partner?.nukePower ?? 0,
    gdp:partner?.gdp ?? 0,
    actualGdp:partner?.gdpUsd ?? 0,
    growth:partner?.growth ?? 0,
    inflation:partner?.inflation ?? 0,
    unemployment:partner?.unemployment ?? 0,
    currencyName:partner?.currencyName || '',
    currencyCode:partner?.currencyCode || '',
    currencyRate:partner?.currencyRate || 1,
    hostility:partner?.hostility ?? Math.max(0, 100 - (partner?.relation ?? 50)),
    nuclearScars:partner?.nuclearScars || 0,
    label:partner?.relation >= 72 ? '協調' : partner?.relation >= 48 ? '警戒' : '対立',
  };
}

function formatDebugWorldCoords(country){
  if(!country) return '-';
  const latText = typeof country.lat === 'number' ? `${Math.abs(country.lat).toFixed(2)}°${country.lat >= 0 ? 'N' : 'S'}` : '-';
  const lonText = typeof country.lon === 'number' ? `${Math.abs(country.lon).toFixed(2)}°${country.lon >= 0 ? 'E' : 'W'}` : '-';
  return `${latText} / ${lonText} / x${country.cx.toFixed(3)} y${country.cy.toFixed(3)}`;
}

function getGeoConfig(mode=S.currentMapMode){
  if(mode === 'world') return {minLon:-180, maxLon:180, maxLat:85, minLat:-60};
  return {minLon:122, maxLon:154, maxLat:48, minLat:20};
}

function normalizedToGeo(mode, nx, ny){
  const cfg = getGeoConfig(mode);
  return {
    lon:cfg.minLon + clamp(nx, 0, 1) * (cfg.maxLon - cfg.minLon),
    lat:cfg.maxLat - clamp(ny, 0, 1) * (cfg.maxLat - cfg.minLat),
  };
}

function geoToNormalized(mode, lat, lon){
  const cfg = getGeoConfig(mode);
  return {
    x:clamp((lon - cfg.minLon) / Math.max(1e-9, cfg.maxLon - cfg.minLon), 0, 1),
    y:clamp((cfg.maxLat - lat) / Math.max(1e-9, cfg.maxLat - cfg.minLat), 0, 1),
  };
}

let prefectureGeoOverridesApplied = false;
function applyPrefectureGeoOverrides(){
  if(prefectureGeoOverridesApplied) return;
  Object.entries(JAPAN_COMPANY_COORD_OVERRIDES).forEach(([prefectureId, geo]) => {
    const prefecture = prefectureMap[prefectureId];
    if(!prefecture) return;
    const mapped = geoToNormalized('japan', geo.lat, geo.lon);
    prefecture.x = mapped.x;
    prefecture.y = mapped.y;
  });
  syncRegionCentersToPrefectures();
  prefectureGeoOverridesApplied = true;
}

function formatLonLabel(value){
  return `${Math.abs(value).toFixed(1)}°${value >= 0 ? 'E' : 'W'}`;
}

function formatLatLabel(value){
  return `${Math.abs(value).toFixed(1)}°${value >= 0 ? 'N' : 'S'}`;
}

function drawMapRulers(mode){
  if(mode === 'space') return;
  if(mode === 'japan') return;
  const viewport = getMapViewport(mode);
  const rulerTop = 24;
  const rulerLeft = 58;
  const verticalTicks = 6;
  const horizontalTicks = 5;
  const showGeoLabels = mode === 'world';
  ctx.save();
  ctx.fillStyle = mode === 'world' ? 'rgba(4,15,7,.92)' : 'rgba(7,14,17,.88)';
  ctx.fillRect(viewport.x, viewport.y, viewport.w, rulerTop);
  ctx.fillRect(viewport.x, viewport.y, rulerLeft, viewport.h);

  ctx.strokeStyle = mode === 'world' ? 'rgba(109,255,170,.28)' : 'rgba(229,242,233,.16)';
  ctx.lineWidth = 1;
  for(let i = 0; i <= verticalTicks; i++){
    const x = viewport.x + viewport.w * (i / verticalTicks);
    const world = canvasToWorld(x, viewport.y + rulerTop + 2);
    const geo = normalizedToGeo(mode, world.x, world.y);
    ctx.beginPath();
    ctx.moveTo(x, viewport.y + rulerTop - 7);
    ctx.lineTo(x, viewport.y + rulerTop);
    ctx.stroke();
    if(showGeoLabels){
      ctx.fillStyle = 'rgba(216,255,231,.88)';
      ctx.font = '10px sans-serif';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText(formatLonLabel(geo.lon), x, viewport.y + rulerTop * 0.5);
    }
  }
  for(let i = 0; i <= horizontalTicks; i++){
    const y = viewport.y + viewport.h * (i / horizontalTicks);
    const world = canvasToWorld(viewport.x + rulerLeft + 2, y);
    const geo = normalizedToGeo(mode, world.x, world.y);
    ctx.beginPath();
    ctx.moveTo(viewport.x + rulerLeft - 7, y);
    ctx.lineTo(viewport.x + rulerLeft, y);
    ctx.stroke();
    if(showGeoLabels){
      ctx.save();
      ctx.translate(viewport.x + rulerLeft * 0.52, y);
      ctx.rotate(-Math.PI / 2);
      ctx.fillStyle = 'rgba(216,255,231,.88)';
      ctx.font = '10px sans-serif';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText(formatLatLabel(geo.lat), 0, 0);
      ctx.restore();
    }
  }

  if(S.mapCursorWorld && S.mapCursorWorld.x >= 0 && S.mapCursorWorld.x <= 1 && S.mapCursorWorld.y >= 0 && S.mapCursorWorld.y <= 1){
    const pos = worldToCanvas(S.mapCursorWorld.x, S.mapCursorWorld.y);
    const geo = normalizedToGeo(mode, S.mapCursorWorld.x, S.mapCursorWorld.y);
    if(showGeoLabels){
      ctx.strokeStyle = 'rgba(109,255,170,.5)';
      ctx.lineWidth = 1;
      ctx.beginPath();
      ctx.moveTo(pos.x, viewport.y + rulerTop - 6);
      ctx.lineTo(pos.x, viewport.y + rulerTop + 2);
      ctx.moveTo(viewport.x + rulerLeft - 6, pos.y);
      ctx.lineTo(viewport.x + rulerLeft + 2, pos.y);
      ctx.stroke();

      ctx.fillStyle = 'rgba(5,18,10,.96)';
      ctx.strokeStyle = 'rgba(109,255,170,.35)';
      ctx.lineWidth = 1;
      ctx.beginPath();
      ctx.roundRect(pos.x - 34, viewport.y + 3, 68, 17, 8);
      ctx.fill();
      ctx.stroke();
      ctx.fillStyle = '#f7fff9';
      ctx.font = '10px sans-serif';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText(formatLonLabel(geo.lon), pos.x, viewport.y + 11.5);

      ctx.beginPath();
      ctx.roundRect(viewport.x + 4, pos.y - 8.5, 50, 17, 8);
      ctx.fillStyle = 'rgba(5,18,10,.96)';
      ctx.fill();
      ctx.strokeStyle = 'rgba(109,255,170,.35)';
      ctx.stroke();
      ctx.fillStyle = '#f7fff9';
      ctx.fillText(formatLatLabel(geo.lat), viewport.x + 29, pos.y);
    }
  }
  ctx.restore();
}

function isPlacementPanGesture(){
  return !!(S.mapDragging && lastMapPointerDown && lastMapPointerDown.button === 2 && (S.draggingWork || S.draggingRoute || S.draggingDisaster || S.draggingStrike));
}

function getFocusedMapLabel(){
  const t = currentI18n();
  const mapText = t.map || {};
  if(S.prefectureEnhancementBrush?.active){
    return `${mapText.placing || 'Placing'}: ${S.prefectureEnhancementBrush.type === 'population' ? '人口成長投資' : '県経済投資'}`;
  }
  if(S.mapTraceActive && S.currentMapMode === 'japan'){
    return `${mapText.focus || 'Focus'}: 座標トレース`;
  }
  if(S.draggingWork){
    const config = publicWorksCatalog[S.draggingWork.type];
    if(config) return `${mapText.placing || 'Placing'}: ${config.name}`;
  }
  if(S.draggingDisaster){
    const chaos = typeof getChaosCatalog === 'function' ? getChaosCatalog()[S.draggingDisaster.type] : null;
    if(chaos) return `${mapText.placing || 'Placing'}: ${chaos.name}`;
  }
  if(S.draggingRoute){
    const route = transportNetworkCatalog[S.draggingRoute.type];
    if(route){
      return `${mapText.placing || 'Placing'}: ${route.name}${S.draggingRoute.fromPrefectureId ? ` / ${prefectureMap[S.draggingRoute.fromPrefectureId]?.name || ''}` : ''}`;
    }
  }
  if(S.currentMapMode === 'space'){
    const body = spaceBodies.find(item => item.id === S.spaceFocusBody);
    return body ? `${mapText.focus || 'Focus'}: ${body.name}` : (mapText.focusSpace || 'Focus: Space');
  }
  if(S.currentMapMode === 'world'){
    return `${mapText.focus || 'Focus'}: ${getWorldCountry(S.worldFocusCountry).label}`;
  }
  if(S.japanFocusPrefecture && prefectureMap[S.japanFocusPrefecture]){
    return `${mapText.focus || 'Focus'}: ${prefectureMap[S.japanFocusPrefecture].name}`;
  }
  return mapText.focusJapan || 'Focus: Japan';
}

function updateMapHud(){
  const modeMap = {
    japan:currentI18n().map?.japan || 'Japan Map',
    world:currentI18n().map?.world || 'World Map',
    space:currentI18n().map?.space || 'Space Map',
  };
  const hud = document.getElementById('mapHud');
  const modeEl = document.getElementById('mapHudMode');
  const focusEl = document.getElementById('mapHudFocus');
  const zoomEl = document.getElementById('mapHudZoom');
  const japanBtn = document.getElementById('mapJapanBtn');
  const worldBtn = document.getElementById('mapWorldBtn');
  const spaceBtn = document.getElementById('mapSpaceBtn');
  const traceBtn = document.getElementById('mapTraceBtn');
  const traceRow = document.getElementById('mapTraceRow');
  const foldBtn = document.getElementById('mapHudFoldBtn');
  const collapsed = !!S.mobileMapHudCollapsed;
  if(modeEl) modeEl.textContent = modeMap[S.currentMapMode] || currentI18n().map?.mapGeneric || 'Map';
  if(focusEl) focusEl.textContent = getFocusedMapLabel();
  if(zoomEl) zoomEl.textContent = `Zoom x${S.mapZoom.toFixed(2)}`;
  if(japanBtn) japanBtn.classList.toggle('active', S.currentMapMode === 'japan');
  if(worldBtn) worldBtn.classList.toggle('active', S.currentMapMode === 'world');
  if(spaceBtn) spaceBtn.classList.toggle('active', S.currentMapMode === 'space');
  if(traceBtn){
    traceBtn.style.display = S.currentMapMode === 'japan' ? 'inline-flex' : 'none';
    traceBtn.classList.toggle('active', !!S.mapTraceOpen);
  }
  if(traceRow) traceRow.style.display = (S.currentMapMode === 'japan' && S.mapTraceOpen) ? 'flex' : 'none';
  if(hud){
    hud.classList.toggle('collapsed', collapsed);
    hud.classList.toggle('trace-open', S.currentMapMode === 'japan' && !!S.mapTraceOpen);
  }
  if(foldBtn){
    foldBtn.style.display = 'inline-flex';
    foldBtn.textContent = collapsed ? '▸' : '◂';
    foldBtn.title = S.language === 'en'
      ? (collapsed ? 'Open map controls' : 'Collapse map controls')
      : (collapsed ? 'マップ操作を開く' : 'マップ操作をたたむ');
  }
  document.body.classList.toggle('mode-japan-map', S.currentMapMode === 'japan');
  document.body.classList.toggle('mode-world-map', S.currentMapMode === 'world');
  document.body.classList.toggle('mode-space-map', S.currentMapMode === 'space');
  positionMapHud();
  updateMapTracePanel();
}

function formatMapTracePoint(world){
  const geo = normalizedToGeo('japan', world.x, world.y);
  return {
    x:world.x,
    y:world.y,
    lat:geo.lat,
    lon:geo.lon,
    text:`${geo.lat.toFixed(2)} ${geo.lon.toFixed(2)}`,
  };
}

function updateMapTracePanel(){
  const traceCursor = document.getElementById('mapTraceCursor');
  const traceCount = document.getElementById('mapTracePointCount');
  const traceOutput = document.getElementById('mapTraceOutput');
  const traceHint = document.getElementById('mapTraceHint');
  const traceActiveBtn = document.getElementById('mapTraceActiveBtn');
  if(traceCursor){
    if(S.currentMapMode === 'japan' && S.mapCursorWorld){
      const point = formatMapTracePoint(S.mapCursorWorld);
      traceCursor.textContent = `カーソル: ${point.lat.toFixed(2)} / ${point.lon.toFixed(2)}`;
    } else {
      traceCursor.textContent = 'カーソル: --';
    }
  }
  if(traceCount) traceCount.textContent = `${(S.mapTracePoints || []).length}点`;
  if(traceActiveBtn) traceActiveBtn.textContent = S.mapTraceActive ? 'なぞりON' : 'なぞりOFF';
  if(traceHint){
    traceHint.textContent = S.mapTraceActive
      ? '左ドラッグで日本地図をなぞると、緯度経度の点列をそのままコピーできます。'
      : '座標ツールを開くと、左ドラッグで日本地図をなぞって緯度経度の点列を取れます。';
  }
  if(traceOutput){
    traceOutput.value = (S.mapTracePoints || []).map(point => point.text).join('\n');
  }
}

window.toggleMapTracePanel = function(){
  if(S.currentMapMode !== 'japan') return;
  S.mapTraceOpen = !S.mapTraceOpen;
  if(!S.mapTraceOpen){
    S.mapTraceActive = false;
    S.mapTraceDrawing = false;
  }
  updateMapHud();
  drawMap();
};

window.toggleMapTraceDrawing = function(){
  if(S.currentMapMode !== 'japan') return;
  S.mapTraceOpen = true;
  S.mapTraceActive = !S.mapTraceActive;
  S.mapTraceDrawing = false;
  updateMapTracePanel();
  updateMapHud();
  drawMap();
};

window.clearMapTrace = function(){
  S.mapTracePoints = [];
  S.mapTraceDrawing = false;
  updateMapTracePanel();
  drawMap();
};

window.copyMapTrace = async function(){
  const text = (S.mapTracePoints || []).map(point => point.text).join('\n');
  if(!text){
    addEvent('❌ コピーする座標がありません', 'bad');
    return;
  }
  try{
    if(navigator.clipboard?.writeText){
      await navigator.clipboard.writeText(text);
      addEvent('📐 座標列をコピーしました', 'good');
      return;
    }
  } catch(error){}
  const area = document.getElementById('mapTraceOutput');
  if(area){
    area.focus();
    area.select();
    try{
      document.execCommand('copy');
      addEvent('📐 座標列をコピーしました', 'good');
      return;
    } catch(error){}
  }
  addEvent('❌ コピーに失敗しました', 'bad');
};

window.resetMapView = function(){
  const next = getDefaultMapView(S.currentMapMode);
  S.mapOffsetX = next.offsetX;
  S.mapOffsetY = next.offsetY;
  S.mapZoom = next.zoom;
  clampMapView();
  storeCurrentView();
  updateMapHud();
  drawMap();
};

function setMapFocus(mode, options={}){
  S.mapFocus = mode;
  document.body.classList.toggle('focus-map', mode === 'map');
  document.body.classList.toggle('focus-panel', mode === 'panel');
  if(!options.skipResize) resizeCanvas();
}

function getCompanyMarkerLayout(regionKey){
  const region = regions[regionKey];
  const regionCompanies = companies.filter(c => c.region === regionKey);
  return regionCompanies.map((company, index) => {
    const pos = worldToCanvas(company.mapX ?? region.cx, company.mapY ?? region.cy);
    return {
      company,
      x:pos.x,
      y:pos.y,
      radius:Math.max(6, 7.2 * S.mapZoom)
    };
  });
}

function getClosestRegion(nx, ny){
  let closest = null;
  let closestDist = Infinity;
  Object.entries(regions).forEach(([key, region]) => {
    if(region.isSea) return;
    const dist = Math.hypot(nx - region.cx, ny - region.cy);
    if(dist < closestDist){
      closestDist = dist;
      closest = key;
    }
  });
  return closest;
}

function isNearRiver(nx, ny, threshold=0.04){
  return riverPolylines.some(polyline => polyline.some(point => Math.hypot(nx - point[0], ny - point[1]) <= threshold));
}

function worldDistance(ax, ay, bx, by){
  return Math.hypot(ax - bx, ay - by);
}

function isCanvasPointNearViewport(x, y, margin=28){
  const viewport = getMapViewport(S.currentMapMode);
  return x >= viewport.x - margin && x <= viewport.x + viewport.w + margin && y >= viewport.y - margin && y <= viewport.y + viewport.h + margin;
}

function getRegionDisasterCenter(regionKey){
  const region = regions[regionKey];
  if(!region) return {x:0.5, y:0.5};
  const angle = Math.random() * Math.PI * 2;
  const radius = region.isSea ? 0.02 : 0.024;
  return {
    x:region.cx + Math.cos(angle) * radius,
    y:region.cy + Math.sin(angle) * radius,
  };
}

function isOnJapanLand(worldPos){
  if(getReferenceMask('japan')) return isReferenceLand('japan', worldPos.x, worldPos.y, 46);
  return Object.values(japanPolys).some(poly => pointInPolygon(worldPos, poly));
}

function isPrefectureCoastal(prefectureId){
  return coastalPrefectureIds.has(prefectureId);
}

function validateTransportRoutePlacement(fromPrefectureId, toPrefectureId, type){
  const from = prefectureMap[fromPrefectureId];
  const to = prefectureMap[toPrefectureId];
  if(!from || !to) return {ok:false, reason:'都道府県が見つかりません'};
  if(type === 'ferry'){
    if(!isPrefectureCoastal(fromPrefectureId) || !isPrefectureCoastal(toPrefectureId)){
      return {ok:false, reason:'航路連絡は海に面した県どうしのみ接続できます'};
    }
    if(worldDistance(from.x, from.y, to.x, to.y) < 0.038){
      return {ok:false, reason:'航路連絡は海を挟む県どうしに敷設してください'};
    }
  }
  if((type === 'rail' || type === 'expressway') && (fromPrefectureId === 'okinawa' || toPrefectureId === 'okinawa')){
    return {ok:false, reason:'沖縄への接続は航路連絡か航空回廊を使ってください'};
  }
  return {ok:true};
}

function canPlacePublicWork(workId, worldPos){
  const config = publicWorksCatalog[workId];
  if(!config) return {ok:false, reason:'工事が存在しません'};
  if(!isOnJapanLand(worldPos)) return {ok:false, reason:'海上には建設できません'};
  const regionKey = getClosestRegion(worldPos.x, worldPos.y);
  if(!regionKey) return {ok:false, reason:'日本列島の陸地に配置してください'};
  if(config.allowed === 'river' && !isNearRiver(worldPos.x, worldPos.y)) return {ok:false, reason:'ダムは川沿いにしか建設できません'};
  return {ok:true, regionKey};
}

function formatMoneyCompact(value){
  return formatMoneyByOku(value, 1);
}

function moneyCompact(value){
  return formatMoneyCompact(value);
}

function formatMoneyPrecise(value){
  return formatMoneyByOku(value, 2);
}

function formatShortScale(value, digits=1){
  const num = Number(value) || 0;
  const abs = Math.abs(num);
  const sign = num < 0 ? '-' : '';
  const units = [
    {limit:1e21, suffix:'Sx'},
    {limit:1e18, suffix:'Qi'},
    {limit:1e15, suffix:'Qa'},
    {limit:1e12, suffix:'T'},
    {limit:1e9, suffix:'B'},
    {limit:1e6, suffix:'M'},
    {limit:1e3, suffix:'K'},
  ];
  const picked = units.find(unit => abs >= unit.limit);
  if(!picked){
    const precise = abs >= 100 ? 0 : abs >= 10 ? Math.min(1, digits) : digits;
    return `${sign}${abs.toLocaleString(undefined, {maximumFractionDigits:precise})}`;
  }
  const scaled = abs / picked.limit;
  const precise = scaled >= 100 ? 0 : scaled >= 10 ? Math.min(1, digits) : digits;
  return `${sign}${scaled.toFixed(precise)}${picked.suffix}`;
}

function formatJapaneseScaleFromUnit(value, baseUnit, higherUnits=[], digits=1){
  const num = Number(value) || 0;
  const abs = Math.abs(num);
  const sign = num < 0 ? '-' : '';
  let scaled = abs;
  let unit = baseUnit;
  for(const nextUnit of higherUnits){
    if(scaled < 10000) break;
    scaled /= 10000;
    unit = nextUnit;
  }
  const precise = scaled >= 100 ? 0 : scaled >= 10 ? Math.min(1, digits) : digits;
  const body = scaled.toLocaleString(undefined, {
    minimumFractionDigits:0,
    maximumFractionDigits:precise,
  });
  return `${sign}${body}${unit}`;
}

function formatPopulationCompact(valueWan, digits=1){
  if((S.language || 'ja') === 'ja'){
    return formatJapaneseScaleFromUnit(valueWan, '万', ['億', '兆'], digits);
  }
  const people = (Number(valueWan) || 0) * 10000;
  return formatShortScale(people, digits);
}

function formatGdpCompact(valueCho, digits=1, withCurrency=true){
  if((S.language || 'ja') === 'ja'){
    const body = formatJapaneseScaleFromUnit(valueCho, '兆', ['京', '垓'], digits);
    return body;
  }
  const yen = (Number(valueCho) || 0) * 1000000000000;
  const text = formatShortScale(yen, digits);
  return withCurrency ? `¥${text}` : text;
}

function formatSignedCompact(value, formatter, suffix=''){
  const num = Number(value) || 0;
  const body = formatter(Math.abs(num));
  return `${num >= 0 ? '+' : '-'}${body}${suffix}`;
}

const MONEY_DISPLAY_DIVISOR = 1000;

function formatMoneyByOku(value, digits=1){
  const num = Number(value) || 0;
  const scaled = num / MONEY_DISPLAY_DIVISOR;
  if((S.language || 'ja') === 'ja'){
    return `${formatJapaneseScaleFromUnit(scaled, '円', ['万', '億', '兆'], digits)}`;
  }
  return `¥${formatShortScale(scaled, digits)}`;
}

function buildDonutChartSvg(items, options={}){
  const size = options.size || 240;
  const stroke = options.stroke || 26;
  const radius = (size - stroke) / 2;
  const center = size / 2;
  const circumference = 2 * Math.PI * radius;
  const total = Math.max(1, items.reduce((sum, item) => sum + Math.max(0, item.value || 0), 0));
  let offset = 0;
  const segments = items.map(item => {
    const portion = Math.max(0, item.value || 0) / total;
    const length = circumference * portion;
    const html = `<circle cx="${center}" cy="${center}" r="${radius}" fill="none" stroke="${item.color}" stroke-width="${stroke}" stroke-dasharray="${length} ${circumference - length}" stroke-dashoffset="${-offset}" transform="rotate(-90 ${center} ${center})"></circle>`;
    offset += length;
    return html;
  }).join('');
  const centerTop = options.centerTop || (S.language === 'en' ? 'Weekly' : '週次');
  const centerBottom = options.centerBottom || formatMoneyCompact(total);
  return `<svg class="finance-donut" viewBox="0 0 ${size} ${size}" preserveAspectRatio="xMidYMid meet">
    <circle cx="${center}" cy="${center}" r="${radius}" fill="none" stroke="rgba(255,255,255,.08)" stroke-width="${stroke}"></circle>
    ${segments}
    <circle cx="${center}" cy="${center}" r="${radius - stroke * 0.72}" fill="rgba(7,12,22,.95)"></circle>
    <text x="${center}" y="${center - 6}" text-anchor="middle" fill="#93aac7" font-size="14" font-weight="600">${centerTop}</text>
    <text x="${center}" y="${center + 18}" text-anchor="middle" fill="#f7fbff" font-size="20" font-weight="800">${centerBottom}</text>
  </svg>`;
}

function getFinanceBreakdownGroups(){
  const b = S.weeklyBreakdown || {};
  const income = [
    {label:S.language === 'en' ? 'Tax' : '税収', value:Math.max(0, b.tax || 0), color:'#4ecca3'},
    {label:S.language === 'en' ? 'Trade' : '貿易収支', value:Math.max(0, b.trade || 0), color:'#4fc3f7'},
    {label:S.language === 'en' ? 'Dividend' : '配当', value:Math.max(0, b.dividend || 0), color:'#ffd54f'},
    {label:S.language === 'en' ? 'Space' : '宇宙収入', value:Math.max(0, b.space || 0), color:'#9575cd'},
    {label:S.language === 'en' ? 'Public Revenue' : '公共収入', value:Math.max(0, b.publicRevenue || 0), color:'#7fd1ff'},
    {label:S.language === 'en' ? 'Adjustments' : '調整', value:Math.max(0, b.adjustments || 0), color:'#26a69a'},
  ].filter(item => item.value > 0);
  const expenses = [
    {label:S.language === 'en' ? 'Budget' : '予算支出', value:Math.max(0, b.budget || 0), color:'#ef5350'},
    {label:S.language === 'en' ? 'Company Support' : '企業支援', value:Math.max(0, b.companySupport || 0), color:'#ff8a65'},
    {label:S.language === 'en' ? 'Operations' : '公共運転費', value:Math.max(0, b.operations || 0), color:'#90a4ae'},
    {label:S.language === 'en' ? 'Social Security' : '社会保障', value:Math.max(0, b.socialSecurity || 0), color:'#f06292'},
    {label:S.language === 'en' ? 'Healthcare' : '医療', value:Math.max(0, b.healthcare || 0), color:'#5c6bc0'},
    {label:S.language === 'en' ? 'Childcare' : '子育て', value:Math.max(0, b.childcare || 0), color:'#26a69a'},
    {label:S.language === 'en' ? 'Administration' : '行政費', value:Math.max(0, b.administration || 0), color:'#b0bec5'},
    {label:S.language === 'en' ? 'Interest' : '利払い', value:Math.max(0, b.interest || 0), color:'#ab47bc'},
    {label:S.language === 'en' ? 'Principal' : '元本返済', value:Math.max(0, b.principal || 0), color:'#8d6e63'},
    {label:S.language === 'en' ? 'Trade Loss' : '貿易赤字', value:Math.max(0, -(b.trade || 0)), color:'#26c6da'},
    {label:S.language === 'en' ? 'Adjustments' : '調整', value:Math.max(0, -(b.adjustments || 0)), color:'#00897b'},
  ].filter(item => item.value > 0);
  return {income, expenses};
}

function getWeeklyMoneyNet(){
  const b = S.weeklyBreakdown || {};
  return (b.tax || 0)
    + (b.trade || 0)
    + (b.dividend || 0)
    + (b.space || 0)
    + (b.publicRevenue || 0)
    + (b.adjustments || 0)
    - (b.budget || 0)
    - (b.companySupport || 0)
    - (b.operations || 0)
    - (b.socialSecurity || 0)
    - (b.healthcare || 0)
    - (b.childcare || 0)
    - (b.administration || 0)
    - (b.interest || 0)
    - (b.principal || 0);
}

function renderFinanceBreakdownPanel(){
  const {income, expenses} = getFinanceBreakdownGroups();
  const incomeTotal = income.reduce((sum, item) => sum + item.value, 0);
  const expenseTotal = expenses.reduce((sum, item) => sum + item.value, 0);
  const net = incomeTotal - expenseTotal;
  const incomeHtml = income.length ? income.map(item => `<div class="finance-legend-row"><span class="finance-legend-swatch" style="background:${item.color}"></span><span>${item.label}</span><span>${((item.value / Math.max(1, incomeTotal)) * 100).toFixed(1)}%</span><strong>${formatMoneyPrecise(item.value)}</strong></div>`).join('') : `<div class="compact-note">${S.language === 'en' ? 'No positive inflow this week.' : '今週の収入項目はありません。'}</div>`;
  const expenseHtml = expenses.length ? expenses.map(item => `<div class="finance-legend-row"><span class="finance-legend-swatch" style="background:${item.color}"></span><span>${item.label}</span><span>${((item.value / Math.max(1, expenseTotal)) * 100).toFixed(1)}%</span><strong>${formatMoneyPrecise(item.value)}</strong></div>`).join('') : `<div class="compact-note">${S.language === 'en' ? 'No outflow this week.' : '今週の支出項目はありません。'}</div>`;
  return `<div class="section">
    <h3>${S.language === 'en' ? '💰 Fiscal Dashboard' : '💰 財政ダッシュボード'}</h3>
    <div class="compact-note">${S.language === 'en' ? 'Tap the money card in the top bar to inspect weekly inflow and outflow with detailed composition.' : '上のお金カードから、週次の収入と支出の構成を円グラフで確認できます。'}</div>
    <div class="finance-summary-grid">
      <div class="finance-summary-box"><div class="label">${S.language === 'en' ? 'Treasury' : '国庫'}</div><div class="value">${formatMoneyCompact(S.money)}</div></div>
      <div class="finance-summary-box"><div class="label">${S.language === 'en' ? 'Income' : '収入'}</div><div class="value" style="color:#4ecca3">${formatMoneyCompact(incomeTotal)}</div></div>
      <div class="finance-summary-box"><div class="label">${S.language === 'en' ? 'Expense' : '支出'}</div><div class="value" style="color:#ef5350">${formatMoneyCompact(expenseTotal)}</div></div>
      <div class="finance-summary-box"><div class="label">${S.language === 'en' ? 'Net' : '差引'}</div><div class="value" style="color:${net >= 0 ? '#4ecca3' : '#ef5350'}">${formatMoneyCompact(net)}</div></div>
    </div>
    <div class="finance-grid">
      <div class="finance-card">
        <h3>${S.language === 'en' ? 'Income Breakdown' : '収入内訳'}</h3>
        <div class="finance-total">${formatMoneyCompact(incomeTotal)}</div>
        ${buildDonutChartSvg(income.length ? income : [{label:'none', value:1, color:'rgba(255,255,255,.08)'}], {centerTop:S.language === 'en' ? 'Income' : '収入', centerBottom:formatMoneyCompact(incomeTotal)})}
        <div class="finance-legend">${incomeHtml}</div>
      </div>
      <div class="finance-card">
        <h3>${S.language === 'en' ? 'Expense Breakdown' : '支出内訳'}</h3>
        <div class="finance-total">${formatMoneyCompact(expenseTotal)}</div>
        ${buildDonutChartSvg(expenses.length ? expenses : [{label:'none', value:1, color:'rgba(255,255,255,.08)'}], {centerTop:S.language === 'en' ? 'Expense' : '支出', centerBottom:formatMoneyCompact(expenseTotal)})}
        <div class="finance-legend">${expenseHtml}</div>
      </div>
    </div>
  </div>`;
}

function escapeHtml(value){
  return String(value).replace(/[&<>"']/g, ch => ({
    '&':'&amp;',
    '<':'&lt;',
    '>':'&gt;',
    '"':'&quot;',
    "'":'&#39;',
  }[ch]));
}

function reportRuntimeError(stage, error){
  const message = `${stage}: ${error?.message || error}`;
  if(!window.__reportedRuntimeErrors) window.__reportedRuntimeErrors = new Set();
  if(window.__reportedRuntimeErrors.has(message)) return;
  window.__reportedRuntimeErrors.add(message);
  console.error(`[${stage}]`, error);
  const panel = document.getElementById('panelContent');
  if(panel && !panel.innerHTML.trim()){
    panel.innerHTML = `<div class="section"><h3>⚠ エラー</h3><div class="compact-note">${escapeHtml(message)}</div></div>`;
  }
  const log = document.getElementById('eventLog');
  if(log){
    const div = document.createElement('div');
    div.className = 'event-msg bad';
    div.textContent = message;
    log.prepend(div);
  }
}

let pendingPanelRender = 0;
let pendingMapDraw = 0;
let pendingHudRefresh = 0;
let viewLoaderToken = 0;
let viewLoaderHideTimer = 0;

function schedulePanelRender(){
  if(pendingPanelRender) return;
  pendingPanelRender = requestAnimationFrame(() => {
    pendingPanelRender = 0;
    try{ renderPanel(); }catch(error){ reportRuntimeError('renderPanel', error); }
  });
}

function scheduleMapDraw(){
  if(pendingMapDraw) return;
  pendingMapDraw = requestAnimationFrame(() => {
    pendingMapDraw = 0;
    try{ drawMap(); }catch(error){ reportRuntimeError('drawMap', error); }
  });
}

function scheduleMapHudRefresh(){
  if(pendingHudRefresh) return;
  pendingHudRefresh = requestAnimationFrame(() => {
    pendingHudRefresh = 0;
    try{ updateMapHud(); }catch(error){ reportRuntimeError('updateMapHud', error); }
  });
}

function cancelQueuedViewWork(){
  if(pendingPanelRender){
    cancelAnimationFrame(pendingPanelRender);
    pendingPanelRender = 0;
  }
  if(pendingMapDraw){
    cancelAnimationFrame(pendingMapDraw);
    pendingMapDraw = 0;
  }
  if(pendingHudRefresh){
    cancelAnimationFrame(pendingHudRefresh);
    pendingHudRefresh = 0;
  }
}

function disposePanelDom(){
  const panel = document.getElementById('panelContent');
  if(!panel) return;
  panel.replaceChildren();
  panel.scrollTop = 0;
}

function disposeModalDom(){
  const title = document.getElementById('modalTitle');
  const content = document.getElementById('modalContent');
  if(title) title.textContent = '';
  if(content) content.replaceChildren();
}

function performPeriodicUiMaintenance(){
  S.lastUiMaintenanceWeek = S.totalWeeks || 0;
  cancelQueuedViewWork();
  hideTooltip();
  disposePanelDom();
  const dockLogs = [
    document.getElementById('eventLog'),
  ];
  dockLogs.forEach(log => {
    if(!log) return;
    while(log.children.length > 18){
      log.removeChild(log.lastChild);
    }
  });
  if(Array.isArray(S.worldLog) && S.worldLog.length > 24) S.worldLog = S.worldLog.slice(0, 24);
  if(Array.isArray(S.eventHistory) && S.eventHistory.length > 180) S.eventHistory = S.eventHistory.slice(0, 180);
  schedulePanelRender();
  scheduleMapDraw();
  scheduleMapHudRefresh();
  addEvent(S.language === 'en' ? '♻ Periodic UI maintenance completed' : '♻ 定期UIメンテナンスを実施', '');
}

function maybePerformPeriodicUiMaintenance(){
  const intervalWeeks = 156;
  if(!S.started || !S.totalWeeks || S.totalWeeks < intervalWeeks) return;
  if((S.lastUiMaintenanceWeek || 0) > 0 && (S.totalWeeks - S.lastUiMaintenanceWeek) < intervalWeeks) return;
  if(S.cyberMiniGame) return;
  const modal = document.getElementById('actionModal');
  if(modal?.classList.contains('show')) return;
  performPeriodicUiMaintenance();
}

function showViewLoader(text=''){
  const loader = document.getElementById('viewLoader');
  const label = document.getElementById('viewLoaderLabel');
  if(!loader || !label) return;
  clearTimeout(viewLoaderHideTimer);
  label.textContent = text || currentI18n().loader;
  loader.classList.add('show');
  document.body.classList.add('ui-loading');
}

function hideViewLoader(token=viewLoaderToken){
  const loader = document.getElementById('viewLoader');
  if(!loader) return;
  clearTimeout(viewLoaderHideTimer);
  viewLoaderHideTimer = setTimeout(() => {
    if(token !== viewLoaderToken) return;
    loader.classList.remove('show');
    document.body.classList.remove('ui-loading');
  }, 90);
}

function runWithViewLoader(text, work){
  const token = ++viewLoaderToken;
  showViewLoader(text);
  setTimeout(() => {
    try{
      work();
    } catch(error){
      reportRuntimeError('ui-switch', error);
    } finally {
      requestAnimationFrame(() => hideViewLoader(token));
    }
  }, 18);
}

const TURN_STEP_OPTIONS = [1,4,13];

function getTurnStepLabel(weeks=S.turnStepWeeks, verbose=false){
  const t = currentI18n().settings || {};
  const labels = {
    1:verbose ? (t.turnStep1 || '1 week') : (S.language === 'en' ? '1w' : '1週'),
    4:verbose ? (t.turnStep4 || '4 weeks') : (S.language === 'en' ? '4w' : '4週'),
    13:verbose ? (t.turnStep13 || '13 weeks') : (S.language === 'en' ? '13w' : '13週'),
  };
  return labels[weeks] || `${weeks}${S.language === 'en' ? 'w' : '週'}`;
}

window.setTurnStep = function(weeks){
  if(!TURN_STEP_OPTIONS.includes(weeks)) return;
  S.turnStepWeeks = weeks;
  updateSpeedButtons();
};

window.cycleTurnStep = function(){
  const currentIndex = Math.max(0, TURN_STEP_OPTIONS.indexOf(S.turnStepWeeks));
  S.turnStepWeeks = TURN_STEP_OPTIONS[(currentIndex + 1) % TURN_STEP_OPTIONS.length];
  updateSpeedButtons();
  if(document.getElementById('actionModal')?.dataset.action === 'settings') openAction('settings');
};

window.setTimeMode = function(mode){
  if(!['realtime','turn'].includes(mode)) return;
  if(mode === 'turn'){
    if(S.speed > 0) S.realtimeSpeed = S.speed;
    S.timeMode = 'turn';
    S.speed = 0;
  } else {
    S.timeMode = 'realtime';
    S.speed = Math.max(1, S.realtimeSpeed || 1);
  }
  updateSpeedButtons();
  if(document.getElementById('actionModal')?.dataset.action === 'settings') openAction('settings');
};

window.setUiMode = function(mode){
  if(!['desktop','mobile'].includes(mode)) return;
  S.uiMode = mode;
  if(mode === 'desktop'){
    S.mobileTopCollapsed = false;
    S.mobileBottomCollapsed = false;
    S.mobilePanelCollapsed = false;
    S.mobileMapHudCollapsed = false;
  }
  scheduleMapHudRefresh();
  updateUI();
  if(document.getElementById('actionModal')?.dataset.action === 'settings') openAction('settings');
};

window.toggleMobileChrome = function(target){
  if(target === 'maphud'){
    S.mobileMapHudCollapsed = !S.mobileMapHudCollapsed;
    scheduleMapHudRefresh();
    updateUI();
    return;
  }
  if(S.uiMode !== 'mobile') return;
  if(target === 'top') S.mobileTopCollapsed = !S.mobileTopCollapsed;
  else if(target === 'bottom') S.mobileBottomCollapsed = !S.mobileBottomCollapsed;
  else if(target === 'panel') S.mobilePanelCollapsed = !S.mobilePanelCollapsed;
  scheduleMapHudRefresh();
  updateUI();
};

window.toggleMobileMapHud = function(){
  S.mobileMapHudCollapsed = !S.mobileMapHudCollapsed;
  updateMapHud();
  updateUI();
};

window.setQualityMode = function(mode){
  if(!['low','balanced','high'].includes(mode)) return;
  S.qualityMode = mode;
  updateUI();
  if(document.getElementById('actionModal')?.dataset.action === 'settings') openAction('settings');
};

function getQualityProfile(){
  if(S.qualityMode === 'low'){
    return {
      animatedFrameMs:180,
      rippleCount:0,
      allowBackdrop:false,
      allowDisasterGlow:false,
      allowWorldTraffic:false,
      allowWarMotion:false,
      allowTransitMotion:false,
    };
  }
  if(S.qualityMode === 'high'){
    return {
      animatedFrameMs:26,
      rippleCount:5,
      allowBackdrop:true,
      allowDisasterGlow:true,
      allowWorldTraffic:true,
      allowWarMotion:true,
      allowTransitMotion:true,
    };
  }
  return {
    animatedFrameMs:52,
    rippleCount:3,
    allowBackdrop:true,
    allowDisasterGlow:true,
    allowWorldTraffic:true,
    allowWarMotion:true,
    allowTransitMotion:true,
  };
}

window.toggleFocusMode = async function(force){
  const next = typeof force === 'boolean' ? force : !S.focusMode;
  S.focusMode = next;
  document.body.classList.toggle('focus-ui', next);
  try{
    if(next && !document.fullscreenElement){
      await document.documentElement.requestFullscreen?.();
    } else if(!next && document.fullscreenElement){
      await document.exitFullscreen?.();
    }
  } catch(error){
    addEvent(S.language === 'en' ? 'ℹ Browser prevented fullscreen. Focus layout only.' : 'ℹ ブラウザが全画面化を拒否しました。レイアウトだけ集中モードにします。', '');
  }
  if(!next) S.focusTrayOpen = false;
  scheduleMapHudRefresh();
  updateUI();
  if(document.getElementById('actionModal')?.dataset.action === 'settings') openAction('settings');
};

document.addEventListener('fullscreenchange', () => {
  const fullscreen = !!document.fullscreenElement;
  if(!fullscreen && S.focusMode){
    S.focusMode = false;
    document.body.classList.remove('focus-ui');
    scheduleMapHudRefresh();
    updateUI();
  }
});

function getFocusTrayStatCatalog(){
  const isEn = S.language === 'en';
  return [
    {id:'money', label:isEn ? 'Treasury' : '国庫'},
    {id:'gdp', label:'GDP'},
    {id:'approval', label:isEn ? 'Approval' : '支持率'},
    {id:'yen', label:isEn ? 'Yen' : '円指数'},
    {id:'tech', label:isEn ? 'Tech' : '技術力'},
    {id:'defense', label:isEn ? 'Defense' : '防衛力'},
    {id:'cyber', label:isEn ? 'Cyber' : 'サイバー'},
    {id:'bubble', label:isEn ? 'Bubble' : 'バブル'},
    {id:'pop', label:isEn ? 'Population' : '人口'},
    {id:'polCap', label:isEn ? 'Political Capital' : '政治資本'},
  ];
}

function getFocusTrayStatValue(statId){
  switch(statId){
    case 'money': return formatMoneyCompact(S.money);
    case 'gdp': return formatGdpCompact(S.gdp, 1);
    case 'approval': return `${S.approval.toFixed(1)}%`;
    case 'yen': return S.yenValue.toFixed(1);
    case 'tech': return S.techLevel.toFixed(1);
    case 'defense': return `${Math.round(S.defPower)}`;
    case 'cyber': return `${Math.round(S.cyberPower)}`;
    case 'bubble': return `${S.bubbleIndex.toFixed(0)}%`;
    case 'pop': return formatPopulationCompact(S.pop, 1);
    case 'polCap': return `${Math.round(S.polCap)}/${S.polCapMax}`;
    default: return '-';
  }
}

function getFocusTrayStatRate(statId){
  const units = currentI18n().units || {};
  const format = (value, suffix='') => {
    const abs = Math.abs(value);
    const text = abs >= 100 ? value.toFixed(0) : abs >= 1 ? value.toFixed(1) : value.toFixed(2);
    return `${value >= 0 ? '+' : ''}${text}${suffix}`;
  };
  switch(statId){
    case 'money': return `${formatSignedCompact(S.money - S.prevMoney, amount => formatMoneyCompact(amount), units.perWeek || '/wk')}`;
    case 'gdp': return `${formatSignedCompact(S.gdp - S.prevGdp, amount => formatGdpCompact(amount, 1), units.perWeek || '/wk')}`;
    case 'approval': return format(S.approval - S.prevApproval, `%${units.perWeek || '/wk'}`);
    case 'yen': return format(S.yenValue - S.prevYenValue, units.perWeek || '/wk');
    case 'tech': return format(S.techLevel - S.prevTechLevel, units.perWeek || '/wk');
    case 'pop': return `${formatSignedCompact(S.pop - S.prevPop, amount => formatPopulationCompact(amount, 1), units.perWeek || '/wk')}`;
    case 'polCap': return format(S.polCap - S.prevPolCap, units.perWeek || '/wk');
    default: return '';
  }
}

function renderFocusTray(){
  const tray = document.getElementById('focusTray');
  const panel = document.getElementById('focusTrayPanel');
  const toggle = document.getElementById('focusTrayToggle');
  if(!tray || !panel || !toggle) return;
  tray.style.display = S.focusMode ? 'flex' : 'none';
  tray.classList.toggle('open', !!S.focusTrayOpen);
  tray.classList.toggle('focus-tray-left', S.focusTrayPosition !== 'bottom');
  tray.classList.toggle('focus-tray-bottom', S.focusTrayPosition === 'bottom');
  const arrow = S.focusTrayPosition === 'bottom' ? (S.focusTrayOpen ? '▼' : '▲') : (S.focusTrayOpen ? '◀' : '▶');
  toggle.textContent = arrow;
  toggle.title = S.language === 'en' ? 'Toggle focus tray' : '集中トレイを開閉';
  const allowed = getFocusTrayStatCatalog();
  const selected = allowed.filter(item => (S.focusTrayStats || []).includes(item.id));
  if(!selected.length){
    panel.innerHTML = `<div class="focus-tray-empty">${S.language === 'en' ? 'No stats selected. Choose them from Settings.' : '表示する統計が選ばれていません。設定から選べます。'}</div>`;
    return;
  }
  panel.innerHTML = selected.map(item => {
    const rate = getFocusTrayStatRate(item.id);
    return `<div class="focus-tray-card" onclick="openTopStatDetail('${item.id}')">
      <div class="lbl">${item.label}</div>
      <div class="val">${getFocusTrayStatValue(item.id)}</div>
      <div class="rate ${rate.startsWith('-') ? 'rate-neg' : 'rate-pos'}">${rate || (S.language === 'en' ? 'Tap to inspect' : '押すと詳細')}</div>
      <div class="hint">${S.language === 'en' ? 'Tap for chart' : '押すとグラフ'}</div>
    </div>`;
  }).join('');
}

function renderFocusTimeDock(){
  const dock = document.getElementById('focusTimeDock');
  if(!dock) return;
  dock.style.display = S.focusMode ? 'flex' : 'none';
  if(!S.focusMode){
    dock.innerHTML = '';
    return;
  }
  const isEn = S.language === 'en';
  const timeTexts = currentI18n().settings || {};
  const realtimeLabel = currentI18n().bottomRealtime || (isEn ? '⏱Realtime' : '⏱リアル');
  const turnLabel = currentI18n().bottomTurn || (isEn ? '🎲Turn' : '🎲ターン');
  const advanceLabel = currentI18n().bottomAdvance || (isEn ? 'Advance' : '進む');
  const dateLabel = isEn ? 'Date' : '日付';
  const weekText = typeof formatWeekText === 'function' ? formatWeekText() : `${S.year}${isEn ? ' W' : '年 第'}${S.week}${isEn ? '' : '週'}`;
  dock.innerHTML = `
    <div class="dock-group">
      <span class="dock-label">${dateLabel}</span>
      <button class="dock-btn active" disabled>${weekText}</button>
    </div>
    <span class="dock-divider"></span>
    <div class="dock-group">
      <span class="dock-label">${isEn ? 'Flow' : '進行'}</span>
      <button class="dock-btn ${S.timeMode === 'realtime' ? 'active' : ''}" onclick="setTimeMode('realtime')">${realtimeLabel}</button>
      <button class="dock-btn ${S.timeMode === 'turn' ? 'btn-turn-active' : ''}" onclick="setTimeMode('turn')">${turnLabel}</button>
    </div>
    <span class="dock-divider"></span>
    <div class="dock-group">
      <span class="dock-label">${isEn ? 'Speed' : '速度'}</span>
      <button class="dock-btn ${S.timeMode === 'realtime' && S.speed === 0 ? 'active' : ''}" onclick="setSpeed(0)" ${S.timeMode !== 'realtime' ? 'disabled' : ''}>⏸</button>
      <button class="dock-btn ${S.timeMode === 'realtime' && S.speed === 1 ? 'active' : ''}" onclick="setSpeed(1)" ${S.timeMode !== 'realtime' ? 'disabled' : ''}>x1</button>
      <button class="dock-btn ${S.timeMode === 'realtime' && S.speed === 2 ? 'active' : ''}" onclick="setSpeed(2)" ${S.timeMode !== 'realtime' ? 'disabled' : ''}>x2</button>
      <button class="dock-btn ${S.timeMode === 'realtime' && S.speed === 4 ? 'active' : ''}" onclick="setSpeed(4)" ${S.timeMode !== 'realtime' ? 'disabled' : ''}>x4</button>
    </div>
    <span class="dock-divider"></span>
    <div class="dock-group">
      <span class="dock-label">${isEn ? 'Turn' : 'ターン'}</span>
      <button class="dock-btn ${S.timeMode === 'turn' && S.turnStepWeeks === 1 ? 'btn-turn-active' : ''}" onclick="setTurnStep(1)" ${S.turnProcessing ? 'disabled' : ''}>${timeTexts.turnStep1 || '1 week'}</button>
      <button class="dock-btn ${S.timeMode === 'turn' && S.turnStepWeeks === 4 ? 'btn-turn-active' : ''}" onclick="setTurnStep(4)" ${S.turnProcessing ? 'disabled' : ''}>${timeTexts.turnStep4 || '4 weeks'}</button>
      <button class="dock-btn ${S.timeMode === 'turn' && S.turnStepWeeks === 13 ? 'btn-turn-active' : ''}" onclick="setTurnStep(13)" ${S.turnProcessing ? 'disabled' : ''}>${timeTexts.turnStep13 || '13 weeks'}</button>
      <button class="dock-btn btn-turn-active" onclick="advanceTurn()" ${S.timeMode !== 'turn' || S.turnProcessing || !S.started ? 'disabled' : ''}>⏭${advanceLabel}</button>
    </div>
  `;
}

function getFloatingDockKeys(name){
  if(name === 'news'){
    return {dock:'newsDock', header:'newsDockHeader', collapse:'newsDockCollapse', pos:'newsDockPos', collapsed:'newsDockCollapsed'};
  }
  return null;
}

function clampFloatingDockPosition(name){
  const keys = getFloatingDockKeys(name);
  if(!keys) return {left:10, top:116};
  const dock = document.getElementById(keys.dock);
  const area = {
    width:window.innerWidth || document.documentElement.clientWidth || 1280,
    height:window.innerHeight || document.documentElement.clientHeight || 720,
  };
  const margin = 8;
  const width = dock?.offsetWidth || 340;
  const height = dock?.offsetHeight || 200;
  const next = S[keys.pos] || {left:10, top:116};
  next.left = clamp(next.left, margin, Math.max(margin, area.width - width - margin));
  next.top = clamp(next.top, margin, Math.max(margin, area.height - height - margin));
  S[keys.pos] = next;
  return next;
}

function positionFloatingDock(name){
  const keys = getFloatingDockKeys(name);
  if(!keys) return;
  const dock = document.getElementById(keys.dock);
  if(!dock) return;
  const pos = clampFloatingDockPosition(name);
  dock.style.left = `${Math.round(pos.left)}px`;
  dock.style.top = `${Math.round(pos.top)}px`;
}

function renderHistoryDock(){
  return;
}

function syncFloatingDockState(){
  ['news'].forEach(name => {
    const keys = getFloatingDockKeys(name);
    const dock = document.getElementById(keys.dock);
    const collapse = document.getElementById(keys.collapse);
    if(!dock || !collapse) return;
    const collapsed = !!S[keys.collapsed];
    dock.classList.toggle('collapsed', collapsed);
    collapse.textContent = collapsed ? '▸' : '▾';
    collapse.title = S.language === 'en'
      ? (collapsed ? 'Expand' : 'Collapse')
      : (collapsed ? '開く' : 'たたむ');
    positionFloatingDock(name);
  });
}

window.toggleFloatingDock = function(name){
  const keys = getFloatingDockKeys(name);
  if(!keys) return;
  S[keys.collapsed] = !S[keys.collapsed];
  syncFloatingDockState();
};

let activeFloatingDockDrag = null;
function initFloatingDocks(){
  if(initFloatingDocks._bound) return;
  initFloatingDocks._bound = true;
  ['news'].forEach(name => {
    const keys = getFloatingDockKeys(name);
    const header = document.getElementById(keys.header);
    const dock = document.getElementById(keys.dock);
    if(!header || !dock) return;
    header.addEventListener('pointerdown', event => {
      const target = event.target;
      if(target.closest('.floating-dock-collapse')) return;
      const rect = dock.getBoundingClientRect();
      activeFloatingDockDrag = {
        name,
        dx:event.clientX - rect.left,
        dy:event.clientY - rect.top,
      };
      dock.setPointerCapture?.(event.pointerId);
    });
  });
  window.addEventListener('pointermove', event => {
    if(!activeFloatingDockDrag) return;
    const keys = getFloatingDockKeys(activeFloatingDockDrag.name);
    S[keys.pos] = {
      left:event.clientX - activeFloatingDockDrag.dx,
      top:event.clientY - activeFloatingDockDrag.dy,
    };
    positionFloatingDock(activeFloatingDockDrag.name);
  });
  window.addEventListener('pointerup', () => {
    activeFloatingDockDrag = null;
  });
}

let activeMapHudDrag = null;
function initMapHudDrag(){
  if(initMapHudDrag._bound) return;
  initMapHudDrag._bound = true;
  const handle = document.getElementById('mapHudDragBtn');
  const hud = document.getElementById('mapHud');
  if(!handle || !hud) return;
  handle.addEventListener('pointerdown', event => {
    const rect = hud.getBoundingClientRect();
    activeMapHudDrag = {
      dx:event.clientX - rect.left,
      dy:event.clientY - rect.top,
      startX:S.mapHudOffset?.x || 0,
      startY:S.mapHudOffset?.y || 0,
    };
    handle.setPointerCapture?.(event.pointerId);
    event.preventDefault();
    event.stopPropagation();
  });
  window.addEventListener('pointermove', event => {
    if(!activeMapHudDrag) return;
    const areaRect = gameArea.getBoundingClientRect();
    const viewport = getMapViewport(S.currentMapMode);
    const scaleX = areaRect.width / Math.max(1, mapW);
    const scaleY = areaRect.height / Math.max(1, mapH);
    const baseLeft = areaRect.left + viewport.x * scaleX + 16;
    const baseTop = areaRect.top + viewport.y * scaleY + 8;
    S.mapHudOffset = {
      x:event.clientX - activeMapHudDrag.dx - baseLeft,
      y:event.clientY - activeMapHudDrag.dy - baseTop,
    };
    positionMapHud();
  });
  window.addEventListener('pointerup', () => {
    activeMapHudDrag = null;
  });
}

window.toggleFocusTray = function(force){
  S.focusTrayOpen = typeof force === 'boolean' ? force : !S.focusTrayOpen;
  renderFocusTray();
  if(document.getElementById('actionModal')?.dataset.action === 'settings') openAction('settings');
};

window.setFocusTrayPosition = function(position){
  if(!['left','bottom'].includes(position)) return;
  S.focusTrayPosition = position;
  renderFocusTray();
  if(document.getElementById('actionModal')?.dataset.action === 'settings') openAction('settings');
};

window.toggleFocusTrayStat = function(statId){
  const current = new Set(S.focusTrayStats || []);
  if(current.has(statId) && current.size > 1) current.delete(statId);
  else current.add(statId);
  S.focusTrayStats = getFocusTrayStatCatalog().map(item => item.id).filter(id => current.has(id));
  renderFocusTray();
  if(document.getElementById('actionModal')?.dataset.action === 'settings') openAction('settings');
};

window.advanceTurn = function(weeks=S.turnStepWeeks){
  if(!S.started || S.turnProcessing) return;
  if(S.timeMode !== 'turn') setTimeMode('turn');
  const stepWeeks = TURN_STEP_OPTIONS.includes(weeks) ? weeks : S.turnStepWeeks;
  S.turnProcessing = true;
  updateSpeedButtons();
  runWithViewLoader((currentI18n().settings || {}).turnLoader || currentI18n().loader, () => {
    try{
      for(let i = 0; i < stepWeeks; i++){
        if(!S.started) break;
        gameTick({skipUI:true, source:'turn'});
      }
    } finally {
      S.turnProcessing = false;
      updateUI();
    }
  });
};

function setPanelTabActive(panel){
  document.querySelectorAll('#panelTabs .btn').forEach(button => {
    button.classList.toggle('active', button.dataset.panel === panel);
  });
}

function getPreferredMapModeForPanel(panel){
  if(panel === 'world') return 'world';
  if(panel === 'space') return 'space';
  return 'japan';
}

function switchPanelView(panel, options={}){
  const nextMode = options.mapMode || getPreferredMapModeForPanel(panel);
  const shouldUseLoader = typeof options.useLoader === 'boolean'
    ? options.useLoader
    : (nextMode !== S.currentMapMode && S.started && !document.getElementById('homeScreen')?.classList.contains('show'));
  const loadingText = options.loadingText
    || (nextMode === 'world'
      ? '世界マップを読込中...'
      : nextMode === 'space'
        ? '宇宙データを読込中...'
        : '日本マップを読込中...');
  const applyViewSwitch = () => {
    cancelQueuedViewWork();
    storeCurrentView();
    if(S.draggingWork && panel !== 'infra') cancelWorkPlacement();
    if(S.draggingRoute && panel !== 'infra') cancelRoutePlacement();
    if(S.draggingDisaster && panel !== 'chaos') cancelChaosPlacement();
    if(S.draggingStrike && panel !== 'nuke') cancelStrikePlacement();
    hideTooltip();
    disposePanelDom();
    S.currentPanel = panel;
    S.currentMapMode = nextMode;
    if(nextMode === 'world' && !S.worldFocusCountry) S.worldFocusCountry = 'japan';
    loadViewForMode(nextMode);
    clampMapView(nextMode);
    setPanelTabActive(panel);
    scheduleMapHudRefresh();
    schedulePanelRender();
    scheduleMapDraw();
  };
  if(shouldUseLoader){
    runWithViewLoader(loadingText, applyViewSwitch);
    return;
  }
  applyViewSwitch();
}

function drawNormalizedImage(image, alpha=1){
  if(!image?.complete || !image.naturalWidth) return false;
  const topLeft = worldToCanvas(0, 0);
  const bottomRight = worldToCanvas(1, 1);
  ctx.save();
  ctx.globalAlpha = alpha;
  ctx.drawImage(image, topLeft.x, topLeft.y, bottomRight.x - topLeft.x, bottomRight.y - topLeft.y);
  ctx.restore();
  return true;
}

function fillNormalizedRect(x, y, width, height, fillStyle, alpha=1){
  const topLeft = worldToCanvas(x, y);
  const bottomRight = worldToCanvas(x + width, y + height);
  ctx.save();
  ctx.globalAlpha = alpha;
  ctx.fillStyle = fillStyle;
  ctx.fillRect(topLeft.x, topLeft.y, bottomRight.x - topLeft.x, bottomRight.y - topLeft.y);
  ctx.restore();
}

function maskReferenceWatermark(kind){
  if(kind === 'japan'){
    fillNormalizedRect(0.77, 0.9, 0.22, 0.1, '#020403', 0.98);
    return;
  }
  if(kind === 'world'){
    return;
  }
}

function getIncidentVisualProfile(label=''){
  const text = String(label || '');
  if(text.includes('台風')) return {icon:'🌀', color:[122,187,255], speed:2.8, rings:5};
  if(text.includes('停電')) return {icon:'⚡', color:[255,214,96], speed:3.2, rings:4};
  if(text.includes('暴動')) return {icon:'🪧', color:[255,122,122], speed:2.6, rings:4};
  if(text.includes('山火事')) return {icon:'🔥', color:[255,116,72], speed:3.4, rings:5};
  if(text.includes('化学')) return {icon:'🏭', color:[255,184,90], speed:2.9, rings:4};
  if(text.includes('隕石')) return {icon:'☄', color:[255,120,92], speed:3.8, rings:6};
  if(text.includes('アンバー')) return {icon:'☣', color:[255,176,104], speed:3.1, rings:5};
  if(text.includes('ベルベット')) return {icon:'🧬', color:[224,132,255], speed:3.1, rings:5};
  if(text.includes('医療救援')) return {icon:'💉', color:[124,240,186], speed:2.4, rings:4};
  if(text.includes('送電網')) return {icon:'🔋', color:[120,214,255], speed:2.5, rings:4};
  if(text.includes('都市防災')) return {icon:'🛰', color:[156,188,255], speed:2.3, rings:4};
  if(text.includes('地震')) return {icon:'🌋', color:[255,126,102], speed:3.0, rings:5};
  return {icon:'⚠', color:[255,138,128], speed:2.7, rings:4};
}

function getIncidentDetailAtCanvas(mx, my, mode=S.currentMapMode){
  let best = null;
  const registerHit = payload => {
    const dist = Math.hypot(mx - payload.canvasX, my - payload.canvasY);
    if(dist > payload.hitRadius) return;
    if(!best || dist < best.dist) best = {...payload, dist};
  };
  if(mode === 'japan'){
    Object.entries(regions).forEach(([regionKey, region]) => {
      if(!region.disaster || region.disaster.weeksLeft <= 0) return;
      const pos = worldToCanvas(region.disaster.x, region.disaster.y);
      registerHit({
        kind:'regional-disaster',
        regionKey,
        type:region.disaster.type,
        severity:region.disaster.severity || 0,
        weeksLeft:region.disaster.weeksLeft || 0,
        x:region.disaster.x,
        y:region.disaster.y,
        radius:region.disaster.radius || 0.04,
        canvasX:pos.x,
        canvasY:pos.y,
        hitRadius:Math.max(18, (region.disaster.radius || 0.04) * Math.min(mapW, mapH) * S.mapZoom * 0.74),
      });
    });
  }
  return best;
}

function openIncidentDetailModal(detail){
  if(!detail) return;
  const modal = document.getElementById('actionModal');
  const modalCard = modal.querySelector('.modal');
  if(modalCard) modalCard.classList.remove('modal-company','modal-world-country','modal-finance','modal-prefecture','modal-cyber');
  const region = regions[detail.regionKey];
  const impactCompanies = companies.filter(company => company.region === detail.regionKey);
  const impactWorks = S.publicWorks.filter(work => work.region === detail.regionKey);
  document.getElementById('modalTitle').textContent = `🌋 ${detail.type}`;
  document.getElementById('modalContent').innerHTML = `<div class="section">
    <div class="detail-grid">
      <div class="dg-item"><div class="dg-label">地域</div><div class="dg-value">${region?.name || detail.regionKey}</div></div>
      <div class="dg-item"><div class="dg-label">強度</div><div class="dg-value">${Math.round((detail.severity || 0) * 100)} / 100</div></div>
      <div class="dg-item"><div class="dg-label">残り</div><div class="dg-value">${detail.weeksLeft}週</div></div>
      <div class="dg-item"><div class="dg-label">影響企業</div><div class="dg-value">${impactCompanies.length}社</div></div>
      <div class="dg-item"><div class="dg-label">影響公共</div><div class="dg-value">${impactWorks.length}件</div></div>
      <div class="dg-item"><div class="dg-label">主影響</div><div class="dg-value">${detail.type.includes('停電') ? '電力・通信' : detail.type.includes('台風') ? '港湾・食品・物流' : detail.type.includes('暴動') ? '小売・金融・治安' : detail.type.includes('山火事') ? '木材・都市機能' : detail.type.includes('化学') ? '工場・病院・通信' : '企業・公共・安定度'}</div></div>
    </div>
    <div class="compact-note" style="margin-top:8px">継続中の災害は毎週、地域企業・公共施設・GDP・支持率・安定度へ継続ダメージを与えます。</div>
  </div>`;
  modal.classList.add('show');
}

function spawnMapEffect(effect){
  if(!Array.isArray(S.mapEffects)) S.mapEffects = [];
  const life = Math.max(6, Math.round(effect.life || 20));
  S.mapEffects.push({
    id:`fx-${Date.now()}-${Math.random().toString(36).slice(2,6)}`,
    mode:effect.mode || S.currentMapMode,
    x:effect.x,
    y:effect.y,
    radius:effect.radius || 0.05,
    icon:effect.icon || '⚠',
    label:effect.label || effect.name || effect.type || '',
    color:Array.isArray(effect.color) ? effect.color : [255, 138, 128],
    life,
    maxLife:life,
    visualWeeks:effect.visualWeeks,
  });
  if(S.mapEffects.length > 36) S.mapEffects.shift();
}

function getHoveredMapIncidentAtCanvas(mx, my, mode=S.currentMapMode){
  let best = null;
  const registerHit = (label, x, y, radius) => {
    if(!label) return;
    const dist = Math.hypot(mx - x, my - y);
    if(dist > radius) return;
    if(!best || dist < best.dist) best = {label, dist};
  };
  if(mode === 'japan'){
    Object.values(regions).forEach(region => {
      if(!region.disaster || region.disaster.weeksLeft <= 0) return;
      const pos = worldToCanvas(region.disaster.x, region.disaster.y);
      const radius = Math.max(16, region.disaster.radius * Math.min(mapW, mapH) * S.mapZoom * 0.7);
      registerHit(region.disaster.type, pos.x, pos.y, radius);
    });
  }
  (S.mapEffects || []).filter(effect => effect.mode === mode).forEach(effect => {
    const pos = worldToCanvas(effect.x, effect.y);
    const radius = Math.max(14, effect.radius * Math.min(mapW, mapH) * S.mapZoom * 0.72);
    registerHit(effect.label, pos.x, pos.y, radius);
  });
  return best;
}

function progressMapEffects(){
  if(!Array.isArray(S.mapEffects)) return;
  S.mapEffects = S.mapEffects.filter(effect => {
    effect.life = Math.max(0, (effect.life || 0) - 1);
    return effect.life > 0;
  });
}

function getEffectAnimationWeeks(effect){
  const totalWeeks = Math.max(1, effect?.totalWeeks || effect?.maxLife || effect?.life || 1);
  const requestedWeeks = Math.round(effect?.visualWeeks || 0);
  if(requestedWeeks > 0) return Math.max(1, Math.min(requestedWeeks, totalWeeks));
  return Math.max(1, Math.min(3, totalWeeks));
}

function getEffectElapsedWeeks(effect){
  const totalWeeks = Math.max(1, effect?.totalWeeks || effect?.maxLife || effect?.life || 1);
  if(effect?.persistent){
    return Math.max(0, totalWeeks - (effect?.life || 0));
  }
  const progress = 1 - (effect?.life || 0) / Math.max(1, effect?.maxLife || effect?.life || 1);
  return Math.max(0, Math.round(progress * totalWeeks));
}

function isEffectStillAnimating(effect){
  return getEffectElapsedWeeks(effect) < getEffectAnimationWeeks(effect);
}

function getMapAnimationClock(){
  const base = (S.totalWeeks || 0) * 1.85;
  const realtimeMoving = S.started && S.timeMode === 'realtime' && (S.speed || 0) > 0;
  const turnMoving = !!S.turnProcessing;
  if(realtimeMoving){
    return base + (performance.now() / 1000) * (0.55 + Math.min(1.8, (S.speed || 1) * 0.28));
  }
  if(turnMoving){
    return base + (performance.now() / 1000) * 0.92;
  }
  return base;
}

function drawMapEffects(mode){
  const now = getMapAnimationClock();
  const quality = getQualityProfile();
  const activeRegionalEffects = mode === 'japan'
    ? Object.values(regions)
      .filter(region => region.disaster && region.disaster.weeksLeft > 0)
      .map(region => ({
        x:region.disaster.x,
        y:region.disaster.y,
        radius:region.disaster.radius,
        label:region.disaster.type,
        maxLife:Math.max(1, region.disaster.weeksLeft),
        life:region.disaster.weeksLeft,
        persistent:true,
        totalWeeks:Math.max(1, region.disaster.totalWeeks || region.disaster.weeksLeft),
      }))
    : [];
  [...activeRegionalEffects, ...((S.mapEffects || []).filter(effect => effect.mode === mode))].forEach(effect => {
    const pos = worldToCanvas(effect.x, effect.y);
    const progress = effect.persistent ? 0.45 : 1 - effect.life / Math.max(1, effect.maxLife);
    const visual = getIncidentVisualProfile(effect.label || effect.type || '');
    const color = effect.color || visual.color || [255, 138, 128];
    const baseRadius = effect.radius * Math.min(mapW, mapH) * S.mapZoom * 0.94;
    const pulseRadius = baseRadius * (0.6 + progress * 1.25);
    const elapsedWeeks = getEffectElapsedWeeks(effect);
    const rippleWeeks = getEffectAnimationWeeks(effect);
    const animating = isEffectStillAnimating(effect);
    const activeWaveStrength = effect.persistent
      ? clamp(1 - elapsedWeeks / Math.max(1, rippleWeeks), 0, 1)
      : clamp(1 - progress, 0, 1);
    ctx.save();
    ctx.globalCompositeOperation = 'lighter';
    if(animating && activeWaveStrength > 0.02 && (quality.allowDisasterGlow || quality.rippleCount > 0)){
      const ringCount = Math.max(1, Math.round(Math.max(visual.rings || 0, quality.rippleCount) * (0.55 + activeWaveStrength * 0.65)));
      const speed = (visual.speed || 2.7) * (0.42 + activeWaveStrength * 0.95);
      for(let ring = 0; ring < ringCount; ring++){
        const phase = ((now * speed) + ring * 0.22 + progress * 0.4) % 1;
        const ringProgress = clamp(phase, 0, 1.2);
        ctx.beginPath();
        ctx.arc(pos.x, pos.y, pulseRadius * (0.24 + ringProgress * 1.08), 0, Math.PI * 2);
        ctx.strokeStyle = `rgba(${color[0]},${color[1]},${color[2]},${Math.max(0.04, (0.22 + activeWaveStrength * 0.26) * (1 - phase) - ring * 0.035)})`;
        ctx.lineWidth = Math.max(1.25, 3.1 * S.mapZoom * (1 - ring * 0.1));
        ctx.stroke();
      }
    }
    if(animating && quality.allowBackdrop && quality.allowDisasterGlow && activeWaveStrength > 0.02){
      const glow = ctx.createRadialGradient(pos.x, pos.y, 0, pos.x, pos.y, pulseRadius * 1.5);
      glow.addColorStop(0, `rgba(${color[0]},${color[1]},${color[2]},${0.12 + activeWaveStrength * 0.18})`);
      glow.addColorStop(1, `rgba(${color[0]},${color[1]},${color[2]},0)`);
      ctx.fillStyle = glow;
      ctx.fillRect(pos.x - pulseRadius * 1.5, pos.y - pulseRadius * 1.5, pulseRadius * 3, pulseRadius * 3);
    }
    ctx.restore();
    if(!animating && !effect.persistent) return;
    ctx.fillStyle = '#fff';
    ctx.globalAlpha = animating ? 1 : 0.78;
    ctx.font = `bold ${Math.max(12, 14 * S.mapZoom)}px sans-serif`;
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(effect.icon || visual.icon, pos.x, pos.y);
    ctx.globalAlpha = 1;
  });
}

function hasAnimatedMapActivity(){
  if((S.mapEffects || []).some(effect => isEffectStillAnimating(effect))) return true;
  const quality = getQualityProfile();
  if(S.currentMapMode === 'japan'){
    if(quality.allowTransitMotion && (S.currentPanel === 'infra' || S.draggingRoute) && (S.transportNetworks || []).some(route => route.status === 'active')) return true;
    return Object.values(regions).some(region => region.disaster && region.disaster.weeksLeft > 0 && isEffectStillAnimating({
      persistent:true,
      life:region.disaster.weeksLeft,
      totalWeeks:Math.max(1, region.disaster.totalWeeks || region.disaster.weeksLeft),
      visualWeeks:region.disaster.visualWeeks || 3,
    }));
  }
  if(S.currentMapMode === 'world'){
    return !!(
      (quality.allowWorldTraffic && (((S.tradeShipments || []).length || (S.spyMissions || []).length)))
      || (quality.allowWarMotion && S.activeWar)
    );
  }
  return false;
}

function drawPoly(pts, fill, stroke, lw=2){
  ctx.beginPath();
  pts.forEach((p,i)=>{
    const {x, y} = worldToCanvas(p[0], p[1]);
    i===0 ? ctx.moveTo(x,y) : ctx.lineTo(x,y);
  });
  ctx.closePath();
  if(fill){ctx.fillStyle=fill;ctx.fill();}
  if(stroke){ctx.strokeStyle=stroke;ctx.lineWidth=lw;ctx.stroke();}
}

function pointInPolygon(point, polygon){
  let inside = false;
  for(let i = 0, j = polygon.length - 1; i < polygon.length; j = i++){
    const xi = polygon[i][0], yi = polygon[i][1];
    const xj = polygon[j][0], yj = polygon[j][1];
    const intersect = ((yi > point.y) !== (yj > point.y))
      && (point.x < (xj - xi) * (point.y - yi) / ((yj - yi) || 1e-9) + xi);
    if(intersect) inside = !inside;
  }
  return inside;
}

function getWorldCountryColor(country){
  if(country.id === 'japan') return '#65ffb0';
  const snapshot = getWorldCountrySnapshot(country);
  if(snapshot.relation >= 72) return '#69b7ff';
  if(snapshot.relation >= 48) return '#ffc356';
  return '#ff6e86';
}

function getWorldCountryTheme(country){
  const ring = getWorldCountryColor(country);
  if(country.id === 'japan'){
    return {ring, fill:'rgba(12,44,26,.92)', chip:'rgba(3,12,7,.88)', glow:'rgba(101,255,176,.22)', text:'#f3fff8', meta:'#a6f1c8'};
  }
  const snapshot = getWorldCountrySnapshot(country);
  if(snapshot.relation >= 72){
    return {ring, fill:'rgba(7,24,38,.9)', chip:'rgba(4,12,18,.88)', glow:'rgba(105,183,255,.18)', text:'#f4fbff', meta:'#b4d8ff'};
  }
  if(snapshot.relation >= 48){
    return {ring, fill:'rgba(38,26,8,.9)', chip:'rgba(20,14,4,.88)', glow:'rgba(255,195,86,.16)', text:'#fff9ef', meta:'#ffd88c'};
  }
  return {ring, fill:'rgba(36,8,12,.9)', chip:'rgba(20,5,8,.88)', glow:'rgba(255,110,134,.16)', text:'#fff2f4', meta:'#ffb3c0'};
}

function pathRoundedRect(x, y, width, height, radius){
  const r = Math.max(0, Math.min(radius, width / 2, height / 2));
  ctx.beginPath();
  ctx.moveTo(x + r, y);
  ctx.lineTo(x + width - r, y);
  ctx.quadraticCurveTo(x + width, y, x + width, y + r);
  ctx.lineTo(x + width, y + height - r);
  ctx.quadraticCurveTo(x + width, y + height, x + width - r, y + height);
  ctx.lineTo(x + r, y + height);
  ctx.quadraticCurveTo(x, y + height, x, y + height - r);
  ctx.lineTo(x, y + r);
  ctx.quadraticCurveTo(x, y, x + r, y);
  ctx.closePath();
}

function drawWorldCountryChip(country, center, theme, focused){
  const primary = country.label;
  const snapshot = getWorldCountrySnapshot(country);
  const meta = country.id === 'japan' ? `GDP ${Math.round(S.gdp)}兆` : `友好 ${snapshot.relation}`;
  const chipX = center.x + (country.chipDx || 0) * S.mapZoom;
  const chipY = center.y + (country.chipDy || -22) * S.mapZoom;
  ctx.save();
  ctx.font = `bold ${Math.max(11, 11.5 * S.mapZoom)}px sans-serif`;
  const primaryWidth = ctx.measureText(primary).width;
  ctx.font = `${Math.max(8, 8.4 * S.mapZoom)}px sans-serif`;
  const metaWidth = ctx.measureText(meta).width;
  const dotCount = (country.displaySectors || []).slice(0, 3).length;
  const dotsWidth = dotCount ? dotCount * 9 * S.mapZoom + 6 * S.mapZoom : 0;
  const width = Math.max(82 * S.mapZoom, Math.max(primaryWidth, metaWidth) + 18 * S.mapZoom + dotsWidth);
  const height = focused ? 38 * S.mapZoom : 34 * S.mapZoom;
  const x = chipX - width / 2;
  const y = chipY - height / 2;

  if((country.chipDx || 0) || (country.chipDy || 0)){
    ctx.beginPath();
    ctx.moveTo(center.x, center.y);
    ctx.lineTo(chipX, chipY + height * 0.18);
    ctx.strokeStyle = focused ? theme.ring : 'rgba(255,255,255,.18)';
    ctx.lineWidth = Math.max(1.1, 1.5 * S.mapZoom);
    ctx.stroke();
  }

  ctx.shadowColor = theme.glow;
  ctx.shadowBlur = focused ? 20 : 12;
  pathRoundedRect(x, y, width, height, 10 * S.mapZoom);
  ctx.fillStyle = theme.chip;
  ctx.fill();
  ctx.shadowBlur = 0;
  ctx.strokeStyle = focused ? theme.ring : 'rgba(255,255,255,.14)';
  ctx.lineWidth = focused ? 1.9 : 1.2;
  ctx.stroke();

  ctx.textAlign = 'left';
  ctx.textBaseline = 'alphabetic';
  ctx.fillStyle = theme.text;
  ctx.font = `bold ${Math.max(11, 11.4 * S.mapZoom)}px sans-serif`;
  ctx.fillText(primary, x + 10 * S.mapZoom, y + 14 * S.mapZoom);
  ctx.fillStyle = theme.meta;
  ctx.font = `${Math.max(8, 8.4 * S.mapZoom)}px sans-serif`;
  ctx.fillText(meta, x + 10 * S.mapZoom, y + 27 * S.mapZoom);

  (country.displaySectors || []).slice(0, 3).forEach((sectorId, index) => {
    const sector = sectors[sectorId];
    if(!sector) return;
    const dotX = x + width - 12 * S.mapZoom - (dotCount - 1 - index) * 9 * S.mapZoom;
    const dotY = y + height - 10 * S.mapZoom;
    ctx.beginPath();
    ctx.arc(dotX, dotY, Math.max(2.7, 3.2 * S.mapZoom), 0, Math.PI * 2);
    ctx.fillStyle = sector.color;
    ctx.fill();
    ctx.strokeStyle = '#08110b';
    ctx.lineWidth = 1;
    ctx.stroke();
  });
  ctx.restore();
}

function getWorldCountryAtCanvas(mx, my, extra=8){
  return (S.mapWorldCountries || [])
    .map(country => ({country, dist:Math.hypot(mx - country.x, my - country.y)}))
    .filter(entry => entry.dist <= entry.country.radius + extra)
    .sort((a,b) => a.dist - b.dist)[0]?.country || null;
}

function getPrefectureMarkerAtCanvas(mx, my){
  return (S.mapPrefectureMarkers || [])
    .map(prefecture => ({prefecture, dist:Math.hypot(mx - prefecture.canvasX, my - prefecture.canvasY)}))
    .filter(entry => entry.dist <= entry.prefecture.radius)
    .sort((a,b) => a.dist - b.dist)[0]?.prefecture || null;
}

function getPrefecturesInBrushSelection(selection){
  if(!selection) return [];
  const x1 = Math.min(selection.startX, selection.endX);
  const x2 = Math.max(selection.startX, selection.endX);
  const y1 = Math.min(selection.startY, selection.endY);
  const y2 = Math.max(selection.startY, selection.endY);
  return prefectures.filter(prefecture => {
    const pos = worldToCanvas(prefecture.x, prefecture.y);
    return pos.x >= x1 && pos.x <= x2 && pos.y >= y1 && pos.y <= y2;
  });
}

function drawPrefectureEnhancementBrushOverlay(){
  const selection = S.prefectureBrushSelection;
  if(!selection || S.currentMapMode !== 'japan') return;
  const selected = getPrefecturesInBrushSelection(selection);
  const type = S.prefectureEnhancementBrush?.type || 'economy';
  const fill = type === 'population' ? 'rgba(255,166,120,.18)' : 'rgba(111,214,164,.18)';
  const stroke = type === 'population' ? 'rgba(255,176,110,.86)' : 'rgba(111,238,170,.9)';
  ctx.save();
  ctx.fillStyle = fill;
  ctx.strokeStyle = stroke;
  ctx.lineWidth = 2;
  const left = Math.min(selection.startX, selection.endX);
  const top = Math.min(selection.startY, selection.endY);
  const width = Math.abs(selection.endX - selection.startX);
  const height = Math.abs(selection.endY - selection.startY);
  ctx.beginPath();
  ctx.roundRect(left, top, width, height, 18);
  ctx.fill();
  ctx.stroke();
  selected.forEach(prefecture => {
    const pos = worldToCanvas(prefecture.x, prefecture.y);
    ctx.beginPath();
    ctx.arc(pos.x, pos.y, Math.max(11, 13 * S.mapZoom), 0, Math.PI * 2);
    ctx.fillStyle = fill;
    ctx.fill();
    ctx.strokeStyle = stroke;
    ctx.stroke();
  });
  ctx.fillStyle = '#f7fff9';
  ctx.font = `bold ${Math.max(12, 13 * S.mapZoom)}px sans-serif`;
  ctx.textAlign = 'left';
  ctx.textBaseline = 'top';
  ctx.fillText(`${type === 'population' ? '人口成長投資' : '県経済投資'} / ${selected.length}県`, left + 12, top + 10);
  ctx.restore();
}

function drawMapTraceOverlay(){
  if(S.currentMapMode !== 'japan' || !S.mapTraceOpen) return;
  const points = S.mapTracePoints || [];
  ctx.save();
  if(points.length){
    ctx.strokeStyle = 'rgba(106,224,255,.82)';
    ctx.lineWidth = Math.max(1.8, 2.4 * S.mapZoom);
    ctx.beginPath();
    points.forEach((point, index) => {
      const pos = worldToCanvas(point.x, point.y);
      index === 0 ? ctx.moveTo(pos.x, pos.y) : ctx.lineTo(pos.x, pos.y);
    });
    ctx.stroke();
    points.forEach(point => {
      const pos = worldToCanvas(point.x, point.y);
      ctx.beginPath();
      ctx.arc(pos.x, pos.y, Math.max(2.2, 3.2 * S.mapZoom), 0, Math.PI * 2);
      ctx.fillStyle = 'rgba(127,219,255,.95)';
      ctx.fill();
    });
  }
  if(S.mapCursorWorld){
    const pos = worldToCanvas(S.mapCursorWorld.x, S.mapCursorWorld.y);
    ctx.beginPath();
    ctx.arc(pos.x, pos.y, Math.max(5, 6 * S.mapZoom), 0, Math.PI * 2);
    ctx.strokeStyle = 'rgba(127,219,255,.86)';
    ctx.lineWidth = 1.5;
    ctx.stroke();
  }
  ctx.restore();
}

function getQuadraticPoint(start, control, end, t){
  const omt = 1 - t;
  return {
    x:omt * omt * start.x + 2 * omt * t * control.x + t * t * end.x,
    y:omt * omt * start.y + 2 * omt * t * control.y + t * t * end.y,
  };
}

function drawTradeTruckIcon(x, y, angle, color){
  ctx.save();
  ctx.translate(x, y);
  ctx.rotate(angle);
  ctx.fillStyle = color;
  ctx.fillRect(-8, -4, 10, 8);
  ctx.fillStyle = '#cfd8dc';
  ctx.fillRect(2, -3.5, 5.5, 7);
  ctx.fillStyle = '#08111f';
  ctx.beginPath();
  ctx.arc(-4.5, 4.8, 2.2, 0, Math.PI * 2);
  ctx.arc(4.2, 4.8, 2.2, 0, Math.PI * 2);
  ctx.fill();
  ctx.restore();
}

function drawSpyMissionIcon(x, y, angle){
  ctx.save();
  ctx.translate(x, y);
  ctx.rotate(angle);
  ctx.fillStyle = 'rgba(10,6,22,.96)';
  ctx.beginPath();
  ctx.arc(0, 0, 8.5, 0, Math.PI * 2);
  ctx.fill();
  ctx.strokeStyle = '#d2a6ff';
  ctx.lineWidth = 1.4;
  ctx.stroke();
  ctx.fillStyle = '#f7f0ff';
  ctx.font = 'bold 10px sans-serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText('🕵', 0, 0.5);
  ctx.restore();
}

function drawTransitVehicleIcon(type, x, y, angle){
  const iconMap = {rail:'🚆', expressway:'🚗', ferry:'⛴', airlink:'✈'};
  const colorMap = {rail:'#8dd6ff', expressway:'#ffd180', ferry:'#7ce6c4', airlink:'#f8a5ff'};
  ctx.save();
  ctx.translate(x, y);
  ctx.rotate(angle);
  ctx.beginPath();
  ctx.arc(0, 0, 8.5, 0, Math.PI * 2);
  ctx.fillStyle = 'rgba(5,12,9,.9)';
  ctx.fill();
  ctx.strokeStyle = colorMap[type] || '#d7f1ff';
  ctx.lineWidth = 1.3;
  ctx.stroke();
  ctx.fillStyle = '#f7fffb';
  ctx.font = 'bold 10px sans-serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(iconMap[type] || '•', 0, 0.5);
  ctx.restore();
}

function drawMilitaryConvoyIcon(x, y, angle, side='jp'){
  const color = side === 'jp' ? '#7ec9ff' : '#ff8e86';
  ctx.save();
  ctx.translate(x, y);
  ctx.rotate(angle);
  ctx.fillStyle = color;
  ctx.beginPath();
  ctx.moveTo(11, 0);
  ctx.lineTo(-8, -7);
  ctx.lineTo(-4, 0);
  ctx.lineTo(-8, 7);
  ctx.closePath();
  ctx.fill();
  ctx.fillStyle = '#0a1324';
  ctx.fillRect(-6, -2, 7, 4);
  ctx.restore();
}

function drawWorldMap(){
  if(!ctx || !mapW || !mapH || mapW < 4 || mapH < 4) return;
  const quality = getQualityProfile();
  syncWorldReferenceAnchors();
  S.mapWorldCountries = [];
  ctx.fillStyle = '#010302';
  ctx.fillRect(0, 0, mapW, mapH);
  const artLoaded = drawNormalizedImage(mapArt.world, 1);
  if(!artLoaded){
    worldBackdropPolys.forEach(poly => {
      drawPoly(poly, 'rgba(22,154,78,.95)', 'rgba(116,255,170,.55)', 1.1 * S.mapZoom);
    });
  }
  const vignette = ctx.createRadialGradient(mapW * 0.52, mapH * 0.48, 0, mapW * 0.52, mapH * 0.48, mapW * 0.72);
  vignette.addColorStop(0, 'rgba(0,0,0,0)');
  vignette.addColorStop(1, 'rgba(0,0,0,.22)');
  ctx.fillStyle = vignette;
  ctx.fillRect(0, 0, mapW, mapH);

  const japanCountry = getWorldCountry('japan');
  const japanPos = worldToCanvas(japanCountry.cx, japanCountry.cy);
  S.tradeDeals.forEach(deal => {
    const country = worldCountries.find(item => item.partnerIdx === deal.partnerIdx);
    if(!country) return;
    const target = worldToCanvas(country.cx, country.cy);
    const control = {x:(japanPos.x + target.x) / 2, y:Math.min(japanPos.y, target.y) - 50 * S.mapZoom};
    ctx.beginPath();
    ctx.moveTo(japanPos.x, japanPos.y);
    ctx.quadraticCurveTo(control.x, control.y, target.x, target.y);
    ctx.strokeStyle = deal.isSell ? 'rgba(93,255,156,.32)' : 'rgba(255,196,86,.28)';
    ctx.lineWidth = Math.max(1.2, 2 * S.mapZoom);
    ctx.setLineDash([7 * S.mapZoom, 5 * S.mapZoom]);
    ctx.stroke();
    ctx.setLineDash([]);
  });

  if(quality.allowWorldTraffic){
  S.tradeShipments.slice(0, 18).forEach(shipment => {
    const country = worldCountries.find(item => item.partnerIdx === shipment.partnerIdx);
    if(!country) return;
    const foreignPos = worldToCanvas(country.cx, country.cy);
    const start = shipment.isSell ? japanPos : foreignPos;
    const end = shipment.isSell ? foreignPos : japanPos;
    const control = {x:(start.x + end.x) / 2, y:Math.min(start.y, end.y) - 50 * S.mapZoom};
    const t = clamp(1 - shipment.weeksLeft / Math.max(1, shipment.totalWeeks), 0, 1);
    const pos = getQuadraticPoint(start, control, end, t);
    const ahead = getQuadraticPoint(start, control, end, clamp(t + 0.015, 0, 1));
    drawTradeTruckIcon(pos.x, pos.y, Math.atan2(ahead.y - pos.y, ahead.x - pos.x), shipment.isSell ? '#4ecca3' : '#f39c12');
  });
  (S.spyMissions || []).slice(0, 16).forEach(mission => {
    const country = worldCountries.find(item => item.id === mission.countryId);
    if(!country) return;
    const target = worldToCanvas(country.cx, country.cy);
    const control = {x:(japanPos.x + target.x) / 2, y:Math.min(japanPos.y, target.y) - 82 * S.mapZoom};
    ctx.beginPath();
    ctx.moveTo(japanPos.x, japanPos.y);
    ctx.quadraticCurveTo(control.x, control.y, target.x, target.y);
    ctx.strokeStyle = 'rgba(210,166,255,.26)';
    ctx.lineWidth = Math.max(1.2, 1.8 * S.mapZoom);
    ctx.setLineDash([4 * S.mapZoom, 7 * S.mapZoom]);
    ctx.stroke();
    ctx.setLineDash([]);
    const t = clamp(1 - mission.weeksLeft / Math.max(1, mission.totalWeeks), 0, 1);
    const pos = getQuadraticPoint(japanPos, control, target, t);
    const ahead = getQuadraticPoint(japanPos, control, target, clamp(t + 0.018, 0, 1));
    drawSpyMissionIcon(pos.x, pos.y, Math.atan2(ahead.y - pos.y, ahead.x - pos.x));
  });
  }
  if(S.activeWar){
    const warCountry = getWorldCountry(S.activeWar.countryId);
    if(warCountry){
      const warTarget = worldToCanvas(warCountry.cx, warCountry.cy);
      const enemyOriginWorld = {x:clamp(warCountry.cx + 0.08, 0.05, 0.96), y:clamp(warCountry.cy - 0.04, 0.06, 0.94)};
      const enemyOrigin = worldToCanvas(enemyOriginWorld.x, enemyOriginWorld.y);
      const jpControl = {x:(japanPos.x + warTarget.x) / 2, y:Math.min(japanPos.y, warTarget.y) - 70 * S.mapZoom};
      const enemyControl = {x:(enemyOrigin.x + warTarget.x) / 2, y:Math.min(enemyOrigin.y, warTarget.y) - 32 * S.mapZoom};
      ctx.setLineDash([10 * S.mapZoom, 6 * S.mapZoom]);
      ctx.strokeStyle = 'rgba(126,201,255,.34)';
      ctx.lineWidth = Math.max(1.4, 2.4 * S.mapZoom);
      ctx.beginPath();
      ctx.moveTo(japanPos.x, japanPos.y);
      ctx.quadraticCurveTo(jpControl.x, jpControl.y, warTarget.x, warTarget.y);
      ctx.stroke();
      ctx.strokeStyle = 'rgba(255,142,134,.32)';
      ctx.beginPath();
      ctx.moveTo(enemyOrigin.x, enemyOrigin.y);
      ctx.quadraticCurveTo(enemyControl.x, enemyControl.y, warTarget.x, warTarget.y);
      ctx.stroke();
      ctx.setLineDash([]);
      if(quality.allowWarMotion){
      S.activeWar.jpDispatches.forEach(dispatch => {
        const start = dispatch.returning ? warTarget : japanPos;
        const end = dispatch.returning ? japanPos : warTarget;
        const control = {x:(start.x + end.x) / 2, y:Math.min(start.y, end.y) - 70 * S.mapZoom};
        const t = clamp(1 - dispatch.weeksLeft / Math.max(1, dispatch.totalWeeks), 0, 1);
        const pos = getQuadraticPoint(start, control, end, t);
        const ahead = getQuadraticPoint(start, control, end, clamp(t + 0.018, 0, 1));
        drawMilitaryConvoyIcon(pos.x, pos.y, Math.atan2(ahead.y - pos.y, ahead.x - pos.x), 'jp');
      });
      S.activeWar.enemyDispatches.forEach(dispatch => {
        const start = enemyOrigin;
        const end = warTarget;
        const t = clamp(1 - dispatch.weeksLeft / Math.max(1, dispatch.totalWeeks), 0, 1);
        const pos = getQuadraticPoint(start, enemyControl, end, t);
        const ahead = getQuadraticPoint(start, enemyControl, end, clamp(t + 0.018, 0, 1));
        drawMilitaryConvoyIcon(pos.x, pos.y, Math.atan2(ahead.y - pos.y, ahead.x - pos.x), 'enemy');
      });
      }
    }
  }
  if(S.draggingDisaster && S.draggingDisaster.target === 'world'){
    const chaos = getChaosCatalog()[S.draggingDisaster.type];
    const pos = worldToCanvas(S.draggingDisaster.x, S.draggingDisaster.y);
    ctx.globalAlpha = 0.9;
    ctx.beginPath();
    ctx.arc(pos.x, pos.y, Math.max(18, 26 * S.mapZoom), 0, Math.PI * 2);
    ctx.fillStyle = 'rgba(233,69,96,.18)';
    ctx.fill();
    ctx.strokeStyle = '#ff8a80';
    ctx.lineWidth = 2;
    ctx.stroke();
    ctx.fillStyle = '#fff';
    ctx.font = `bold ${Math.max(11, 12 * S.mapZoom)}px sans-serif`;
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(chaos?.icon || '⚠', pos.x, pos.y);
    ctx.globalAlpha = 1;
  }

  worldCountries.forEach(country => {
    const focused = S.worldFocusCountry === country.id;
    const theme = getWorldCountryTheme(country);
    const center = worldToCanvas(country.cx, country.cy);
    if(quality.allowBackdrop){
      const aura = ctx.createRadialGradient(center.x, center.y, 0, center.x, center.y, (focused ? 58 : 42) * S.mapZoom);
      aura.addColorStop(0, theme.glow);
      aura.addColorStop(1, 'rgba(0,0,0,0)');
      ctx.fillStyle = aura;
      ctx.fillRect(center.x - 68 * S.mapZoom, center.y - 68 * S.mapZoom, 136 * S.mapZoom, 136 * S.mapZoom);
    }

    ctx.beginPath();
    ctx.arc(center.x, center.y, focused ? 16 * S.mapZoom : 12 * S.mapZoom, 0, Math.PI * 2);
    ctx.fillStyle = theme.fill;
    ctx.fill();
    ctx.strokeStyle = theme.ring;
    ctx.lineWidth = focused ? 3 : 2;
    ctx.stroke();

    ctx.beginPath();
    ctx.arc(center.x, center.y, focused ? 7 * S.mapZoom : 5.5 * S.mapZoom, 0, Math.PI * 2);
    ctx.fillStyle = '#f7fff9';
    ctx.fill();

    if(focused){
      ctx.beginPath();
      ctx.arc(center.x, center.y, 23 * S.mapZoom, 0, Math.PI * 2);
      ctx.setLineDash([7 * S.mapZoom, 5 * S.mapZoom]);
      ctx.strokeStyle = theme.ring;
      ctx.lineWidth = 1.4;
      ctx.stroke();
      ctx.setLineDash([]);
    }

    drawWorldCountryChip(country, center, theme, focused);

    S.mapWorldCountries.push({...country, x:center.x, y:center.y, radius:Math.max(20, (country.hitRadius || 24) * S.mapZoom)});
  });

  drawMapEffects('world');
  drawMapRulers('world');
}

function drawMap(){
  if(!ctx || !mapW || !mapH || mapW < 4 || mapH < 4) return;
  if(S.currentMapMode === 'space'){
    drawSpaceMap();
    return;
  }
  if(S.currentMapMode === 'world'){
    drawWorldMap();
    return;
  }
  syncJapanReferenceAnchors();
  S.mapCompanyMarkers = [];
  S.mapPrefectureMarkers = [];
  const hideCompaniesForInfra = S.currentPanel === 'infra';
  const activeInfraFilters = S.currentPanel === 'infra'
    ? ((S.draggingRoute?.type || S.draggingWork?.type) ? [S.draggingRoute?.type || S.draggingWork?.type] : getInfraMapFilterList())
    : [];
  const visibleTransportTypes = new Set(activeInfraFilters.filter(id => transportNetworkCatalog[id]));
  const visibleWorkTypes = new Set(activeInfraFilters.filter(id => publicWorksCatalog[id]));
  ctx.fillStyle = '#020403';
  ctx.fillRect(0, 0, mapW, mapH);
  const japanArtLoaded = drawNormalizedImage(mapArt.japan, 0.98);
  if(!japanArtLoaded){
    Object.entries(japanPolys).forEach(([, pts]) => {
      drawPoly(pts, 'rgba(193,230,201,.96)', 'rgba(224,242,227,.32)', 1.2 * S.mapZoom);
    });
  } else if(mapArt.japanAlt?.complete && mapArt.japanAlt.naturalWidth){
    drawNormalizedImage(mapArt.japanAlt, 0.06);
  }
  maskReferenceWatermark('japan');

  riverPolylines.forEach(polyline => {
    ctx.beginPath();
    polyline.forEach((point, index) => {
      const canvasPos = worldToCanvas(point[0], point[1]);
      index === 0 ? ctx.moveTo(canvasPos.x, canvasPos.y) : ctx.lineTo(canvasPos.x, canvasPos.y);
    });
    ctx.strokeStyle = 'rgba(245,255,248,.16)';
    ctx.lineWidth = Math.max(1.2, 2.2 * S.mapZoom);
    ctx.stroke();
  });

  Object.entries(japanPolys).forEach(([key, pts])=>{
    const rk = key === 'chubu' ? 'chubu' : key;
    const reg = regions[rk];
    if(reg?.disaster){
      const epicenter = worldToCanvas(reg.disaster.x, reg.disaster.y);
      const glow = ctx.createRadialGradient(epicenter.x, epicenter.y, 0, epicenter.x, epicenter.y, reg.disaster.radius * Math.min(mapW, mapH) * S.mapZoom * 1.45);
      glow.addColorStop(0, 'rgba(255,112,86,.24)');
      glow.addColorStop(1, 'rgba(255,112,86,0)');
      ctx.fillStyle = glow;
      ctx.fillRect(epicenter.x - 90 * S.mapZoom, epicenter.y - 90 * S.mapZoom, 180 * S.mapZoom, 180 * S.mapZoom);
    }
    if(!japanArtLoaded){
      drawPoly(pts, null, 'rgba(235,247,238,.42)', 1.1 * S.mapZoom);
    }
  });

  Object.entries(regions).forEach(([k,r])=>{
    if(r.isSea) return;
    const {x, y} = worldToCanvas(r.cx, r.cy);
    const showDetail = S.mapZoom >= 1.08 || S.mapFocus === 'map';
    if(showDetail){
      if(r.developed > 0){
        ctx.fillStyle = '#7fc797';
        ctx.font = `${Math.max(7, 8*S.mapZoom)}px sans-serif`;
        ctx.fillText(`Lv${r.developed}`, x, y + 4*S.mapZoom);
      }
      const resCount = r.resources.length;
      r.resources.forEach((res, i)=>{
        const rx = x + (i - resCount/2) * 9*S.mapZoom;
        const ry = y + 14*S.mapZoom;
        ctx.beginPath(); ctx.arc(rx, ry, 2.7*S.mapZoom, 0, Math.PI*2);
        ctx.fillStyle = resources[res] ? resources[res].color : '#fff';
        ctx.fill();
      });
    }
  });

  const showTransportNetworks = S.currentPanel === 'infra' || !!S.draggingRoute;
  (S.transportNetworks || []).forEach(route => {
    if(!showTransportNetworks) return;
    if(visibleTransportTypes.size && !visibleTransportTypes.has(route.type)) return;
    const config = transportNetworkCatalog[route.type];
    const endpoints = getTransportRouteEndpoints(route);
    if(!config || !endpoints.from || !endpoints.to) return;
    const start = worldToCanvas(endpoints.from.x, endpoints.from.y);
    const end = worldToCanvas(endpoints.to.x, endpoints.to.y);
    if(!isCanvasPointNearViewport(start.x, start.y, 90) && !isCanvasPointNearViewport(end.x, end.y, 90)) return;
    ctx.save();
    ctx.strokeStyle = route.status === 'building' ? 'rgba(255,255,255,.22)' : getTransportNetworkColor(route.type);
    ctx.lineWidth = route.type === 'rail' ? Math.max(1.8, 2.8 * S.mapZoom) : Math.max(1.4, 2.1 * S.mapZoom);
    ctx.setLineDash(route.status === 'building' ? [8 * S.mapZoom, 6 * S.mapZoom] : route.type === 'airlink' ? [10 * S.mapZoom, 7 * S.mapZoom] : []);
    ctx.beginPath();
    ctx.moveTo(start.x, start.y);
    const midX = (start.x + end.x) * 0.5;
    const midY = (start.y + end.y) * 0.5 - Math.min(mapH, mapW) * 0.015 * (route.type === 'airlink' ? 2.3 : route.type === 'ferry' ? 1.3 : 0.7);
    ctx.quadraticCurveTo(midX, midY, end.x, end.y);
    ctx.stroke();
    ctx.setLineDash([]);
    if(S.currentPanel === 'infra' || S.mapZoom >= 1.12){
      const center = getQuadraticPoint(start.x, start.y, midX, midY, end.x, end.y, 0.5);
      if(isCanvasPointNearViewport(center.x, center.y, 26)){
        ctx.beginPath();
        ctx.arc(center.x, center.y, Math.max(8, 9.5 * S.mapZoom), 0, Math.PI * 2);
        ctx.fillStyle = 'rgba(3,10,8,.92)';
        ctx.fill();
        ctx.strokeStyle = getTransportNetworkColor(route.type);
        ctx.lineWidth = 1.4;
        ctx.stroke();
        ctx.fillStyle = '#f4fffb';
        ctx.font = `bold ${Math.max(9, 10 * S.mapZoom)}px sans-serif`;
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';
        ctx.fillText(config.icon || '🛤', center.x, center.y + 0.5);
      }
    }
    if(route.status === 'active' && getQualityProfile().allowTransitMotion){
      const animatedVehicles = route.type === 'rail' ? 2 : 1;
      const phaseSeed = ((route.segmentIndex || 0) * 0.17) + ((route.operatingRate || 80) * 0.001);
      for(let vehicleIndex = 0; vehicleIndex < animatedVehicles; vehicleIndex++){
        const phase = (performance.now() * 0.00012 * (route.type === 'airlink' ? 1.8 : route.type === 'rail' ? 1.25 : 1) + phaseSeed + vehicleIndex / animatedVehicles) % 1;
        const pos = getQuadraticPoint(start.x, start.y, midX, midY, end.x, end.y, phase);
        const ahead = getQuadraticPoint(start.x, start.y, midX, midY, end.x, end.y, clamp(phase + 0.02, 0, 1));
        if(!isCanvasPointNearViewport(pos.x, pos.y, 30)) continue;
        drawTransitVehicleIcon(route.type, pos.x, pos.y, Math.atan2(ahead.y - pos.y, ahead.x - pos.x));
      }
    }
    ctx.restore();
  });

  prefectures.forEach(prefecture => {
    const {x, y} = worldToCanvas(prefecture.x, prefecture.y);
    if(!isCanvasPointNearViewport(x, y, 20)) return;
    const focused = S.japanFocusPrefecture === prefecture.id;
    const radius = Math.max(4.5, 5.6 * S.mapZoom);
    ctx.beginPath();
    ctx.arc(x, y, focused ? radius + 1.2 : radius, 0, Math.PI * 2);
    ctx.fillStyle = focused ? 'rgba(255,244,185,.96)' : 'rgba(242,255,246,.82)';
    ctx.fill();
    ctx.strokeStyle = focused ? '#f39c12' : 'rgba(12,28,18,.84)';
    ctx.lineWidth = focused ? 2 : 1.1;
    ctx.stroke();
    S.mapPrefectureMarkers.push({...prefecture, canvasX:x, canvasY:y, radius:radius + 6});
  });

  if(S.draggingRoute){
    prefectures.forEach(prefecture => {
      const anchor = getPrefectureTransitAnchor(prefecture.id);
      if(!anchor) return;
      const pos = worldToCanvas(anchor.x, anchor.y);
      if(!isCanvasPointNearViewport(pos.x, pos.y, 18)) return;
      ctx.beginPath();
      ctx.arc(pos.x, pos.y, Math.max(2.6, 3.2 * S.mapZoom), 0, Math.PI * 2);
      ctx.fillStyle = prefecture.id === S.draggingRoute.fromPrefectureId ? '#ffd180' : 'rgba(236,247,238,.84)';
      ctx.fill();
      ctx.strokeStyle = prefecture.id === S.draggingRoute.fromPrefectureId ? '#f39c12' : 'rgba(9,22,15,.82)';
      ctx.lineWidth = 1;
      ctx.stroke();
    });
  }

  if(!hideCompaniesForInfra){
    Object.keys(regions).forEach(regionKey => {
      getCompanyMarkerLayout(regionKey).forEach(marker => {
        const sec = sectors[marker.company.sector];
        if(!isCanvasPointNearViewport(marker.x, marker.y, 24)) return;
        ctx.beginPath();
        ctx.arc(marker.x, marker.y, marker.radius, 0, Math.PI * 2);
        ctx.fillStyle = sec.color;
        ctx.fill();
        ctx.lineWidth = 2;
        ctx.strokeStyle = '#f5faf6';
        ctx.stroke();
        ctx.fillStyle = '#08111f';
        ctx.font = `bold ${Math.max(9, 11*S.mapZoom)}px sans-serif`;
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';
        ctx.fillText(sec.icon, marker.x, marker.y + 0.5);
        S.mapCompanyMarkers.push(marker);
      });
    });
  }

  // Discovered materials indicators
  if(S.mapZoom >= 1.08 || ['resources','develop'].includes(S.currentPanel)){
    S.discoveredMaterials.forEach((dm, i)=>{
      const mat = [...uniqueMaterials, ...spaceMaterials].find(m => m.id === dm.id);
      if(!mat) return;
      const reg = regions[dm.region];
      if(!reg) return;
      const anchor = worldToCanvas(reg.cx, reg.cy);
      const x = anchor.x + 20 * S.mapZoom;
      const y = anchor.y - 10 * S.mapZoom + i * 12 * S.mapZoom;
      if(!isCanvasPointNearViewport(x, y, 30)) return;
      ctx.fillStyle = mat.color;
      ctx.font = `bold ${8*S.mapZoom}px sans-serif`;
      ctx.fillText(`✦${mat.name}`, x, y);
    });
  }

  S.tradeDeals.forEach(td => {
    const angle = td.partnerIdx * (Math.PI*2 / tradePartners.length) - Math.PI/2;
    const origin = worldToCanvas(0.67, 0.47);
    const viewport = getMapViewport('japan');
    const ex = origin.x + Math.cos(angle) * viewport.w * 0.36 * S.mapZoom;
    const ey = origin.y + Math.sin(angle) * viewport.h * 0.36 * S.mapZoom;
    ctx.beginPath(); ctx.moveTo(origin.x, origin.y); ctx.lineTo(ex, ey);
    ctx.strokeStyle = td.isSell ? 'rgba(141,213,162,.18)' : 'rgba(255,196,120,.18)';
    ctx.lineWidth = 1.6*S.mapZoom;
    ctx.setLineDash([6*S.mapZoom, 4*S.mapZoom]);
    ctx.stroke(); ctx.setLineDash([]);
  });

  if(S.currentPanel === 'infra'){
    S.publicWorks.forEach(work => {
      if(visibleWorkTypes.size && !visibleWorkTypes.has(work.type)) return;
      const config = publicWorksCatalog[work.type];
      const pos = worldToCanvas(work.x, work.y);
      if(!isCanvasPointNearViewport(pos.x, pos.y, 30)) return;
      ctx.beginPath();
      ctx.arc(pos.x, pos.y, 12 * S.mapZoom, 0, Math.PI * 2);
      ctx.fillStyle = 'rgba(8,17,31,.92)';
      ctx.fill();
      ctx.strokeStyle = '#ffd180';
      ctx.lineWidth = 2;
      ctx.stroke();
      ctx.fillStyle = '#fff';
      ctx.font = `bold ${Math.max(10, 12 * S.mapZoom)}px sans-serif`;
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText(config?.icon || '🏗', pos.x, pos.y + 0.5);
      ctx.textBaseline = 'alphabetic';
      ctx.font = `${Math.max(8, 9 * S.mapZoom)}px sans-serif`;
      ctx.fillStyle = 'rgba(255,255,255,.82)';
      ctx.fillText(config?.name || work.type, pos.x, pos.y - 16 * S.mapZoom);
    });
  }

  drawMapEffects('japan');

  if(S.draggingWork){
    const pos = worldToCanvas(S.draggingWork.x, S.draggingWork.y);
    const config = publicWorksCatalog[S.draggingWork.type];
    const valid = canPlacePublicWork(S.draggingWork.type, S.draggingWork).ok;
    ctx.save();
    ctx.fillStyle = 'rgba(233,69,96,.12)';
    ctx.fillRect(0, 0, mapW, mapH);
    Object.entries(japanPolys).forEach(([, pts]) => {
      drawPoly(pts, 'rgba(78,204,163,.12)', 'rgba(78,204,163,.18)', 1.05 * S.mapZoom);
    });
    prefectures.forEach(prefecture => {
      const anchor = getPrefectureTransitAnchor(prefecture.id) || prefecture;
      const canPlace = canPlacePublicWork(S.draggingWork.type, {x:anchor.x, y:anchor.y}).ok;
      const anchorPos = worldToCanvas(anchor.x, anchor.y);
      if(!isCanvasPointNearViewport(anchorPos.x, anchorPos.y, 28)) return;
      ctx.beginPath();
      ctx.arc(anchorPos.x, anchorPos.y, Math.max(10, 13 * S.mapZoom), 0, Math.PI * 2);
      ctx.fillStyle = canPlace ? 'rgba(78,204,163,.24)' : 'rgba(233,69,96,.16)';
      ctx.fill();
      ctx.strokeStyle = canPlace ? 'rgba(78,204,163,.52)' : 'rgba(255,123,145,.38)';
      ctx.lineWidth = 1;
      ctx.stroke();
    });
    if(config?.allowed === 'river'){
      riverPolylines.forEach(polyline => {
        ctx.beginPath();
        polyline.forEach((point, index) => {
          const riverPos = worldToCanvas(point[0], point[1]);
          index === 0 ? ctx.moveTo(riverPos.x, riverPos.y) : ctx.lineTo(riverPos.x, riverPos.y);
        });
        ctx.strokeStyle = 'rgba(120,224,255,.78)';
        ctx.lineWidth = Math.max(3, 5 * S.mapZoom);
        ctx.stroke();
      });
    }
    ctx.restore();
    ctx.globalAlpha = 0.85;
    ctx.beginPath();
    ctx.arc(pos.x, pos.y, 16 * S.mapZoom, 0, Math.PI * 2);
    ctx.fillStyle = valid ? 'rgba(78,204,163,.35)' : 'rgba(233,69,96,.3)';
    ctx.fill();
    ctx.strokeStyle = valid ? '#4ecca3' : '#e94560';
    ctx.lineWidth = 2;
    ctx.stroke();
    ctx.fillStyle = '#fff';
    ctx.font = `bold ${Math.max(10, 12 * S.mapZoom)}px sans-serif`;
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(config?.icon || '🏗', pos.x, pos.y);
    ctx.globalAlpha = 1;
  }
  if(S.draggingRoute){
    const config = transportNetworkCatalog[S.draggingRoute.type];
    const from = getPrefectureTransitAnchor(S.draggingRoute.fromPrefectureId);
    const fromPos = from ? worldToCanvas(from.x, from.y) : null;
    const toPos = worldToCanvas(S.draggingRoute.x, S.draggingRoute.y);
    ctx.save();
    ctx.strokeStyle = getTransportNetworkColor(S.draggingRoute.type);
    ctx.lineWidth = Math.max(2, 2.8 * S.mapZoom);
    ctx.setLineDash([10 * S.mapZoom, 6 * S.mapZoom]);
    ctx.beginPath();
    if(fromPos){
      ctx.moveTo(fromPos.x, fromPos.y);
      ctx.lineTo(toPos.x, toPos.y);
    } else {
      ctx.arc(toPos.x, toPos.y, 14 * S.mapZoom, 0, Math.PI * 2);
    }
    ctx.stroke();
    ctx.setLineDash([]);
    ctx.fillStyle = 'rgba(255,255,255,.92)';
    ctx.font = `bold ${Math.max(10, 12 * S.mapZoom)}px sans-serif`;
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(config?.icon || '🛤', toPos.x, toPos.y);
    ctx.restore();
  }
  if(S.draggingDisaster && S.draggingDisaster.target === 'japan'){
    const chaos = getChaosCatalog()[S.draggingDisaster.type];
    const pos = worldToCanvas(S.draggingDisaster.x, S.draggingDisaster.y);
    ctx.globalAlpha = 0.88;
    ctx.beginPath();
    ctx.arc(pos.x, pos.y, (chaos?.radius || 0.05) * Math.min(mapW, mapH) * S.mapZoom, 0, Math.PI * 2);
    ctx.fillStyle = 'rgba(233,69,96,.18)';
    ctx.fill();
    ctx.strokeStyle = '#ff8a80';
    ctx.lineWidth = 2;
    ctx.stroke();
    ctx.fillStyle = '#fff';
    ctx.font = `bold ${Math.max(10, 12 * S.mapZoom)}px sans-serif`;
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(chaos?.icon || '⚠', pos.x, pos.y);
    ctx.globalAlpha = 1;
  }
  drawPrefectureEnhancementBrushOverlay();
  drawMapTraceOverlay();
  drawMapRulers('japan');
}

function drawSpaceMap(){
  if(!ctx || !mapW || !mapH || mapW < 4 || mapH < 4) return;
  S.mapSpaceMarkers = [];
  const bg = ctx.createLinearGradient(0, 0, 0, mapH);
  bg.addColorStop(0, '#050914');
  bg.addColorStop(.6, '#08152e');
  bg.addColorStop(1, '#03060f');
  ctx.fillStyle = bg;
  ctx.fillRect(0, 0, mapW, mapH);

  for(let i=0;i<180;i++){
    const sx = ((i * 73) % mapW);
    const sy = ((i * 191) % mapH);
    const size = 1 + ((i * 13) % 3);
    ctx.fillStyle = `rgba(255,255,255,${0.2 + ((i * 7) % 60) / 100})`;
    ctx.fillRect(sx, sy, size, size);
  }

  const sunPos = worldToCanvas(0.12, 0.56);
  const earthPos = worldToCanvas(0.3, 0.56);
  ctx.beginPath();
  ctx.arc(sunPos.x, sunPos.y, 26, 0, Math.PI * 2);
  ctx.fillStyle = '#ffd54f';
  ctx.fill();
  ctx.shadowColor = '#ffd54f';
  ctx.shadowBlur = 25;
  ctx.fillRect(sunPos.x - 1, sunPos.y - 1, 2, 2);
  ctx.shadowBlur = 0;
  ctx.beginPath();
  ctx.ellipse(sunPos.x, sunPos.y, Math.abs(earthPos.x - sunPos.x), 54, 0, 0, Math.PI * 2);
  ctx.strokeStyle = 'rgba(255,255,255,.08)';
  ctx.lineWidth = 1;
  ctx.stroke();
  ctx.beginPath();
  ctx.arc(earthPos.x, earthPos.y, 14, 0, Math.PI * 2);
  ctx.fillStyle = '#64b5f6';
  ctx.fill();
  ctx.fillStyle = 'rgba(255,255,255,.85)';
  ctx.font = 'bold 14px sans-serif';
  ctx.textAlign = 'center';
  ctx.fillText('地球', earthPos.x, earthPos.y - 24);
  const spaceLayout = {
    moon:{x:0.39, y:0.48},
    mars:{x:0.62, y:0.31},
    asteroid:{x:0.84, y:0.54},
    europa:{x:0.73, y:0.78},
  };

  spaceBodies.forEach((body, index) => {
    const targetPos = spaceLayout[body.id] || {x:0.4 + index * 0.15, y:0.5};
    const planetPos = worldToCanvas(targetPos.x, targetPos.y);
    const x = planetPos.x;
    const y = planetPos.y;
    ctx.beginPath();
    if(body.id === 'moon'){
      ctx.ellipse(earthPos.x, earthPos.y, Math.abs(planetPos.x - earthPos.x), Math.max(18, Math.abs(planetPos.y - earthPos.y)), 0, 0, Math.PI * 2);
    } else {
      ctx.ellipse(sunPos.x, sunPos.y, Math.abs(planetPos.x - sunPos.x), Math.max(18, Math.abs(planetPos.y - sunPos.y)), 0, 0, Math.PI * 2);
    }
    ctx.strokeStyle = 'rgba(255,255,255,.08)';
    ctx.lineWidth = 1;
    ctx.stroke();

    ctx.beginPath();
    ctx.arc(x, y, 18 + index * 2, 0, Math.PI * 2);
    ctx.fillStyle = body.color;
    ctx.fill();
    if(S.spaceFocusBody === body.id){
      ctx.strokeStyle = '#fff';
      ctx.lineWidth = 3;
      ctx.stroke();
    }
    ctx.fillStyle = 'rgba(255,255,255,.85)';
    ctx.font = 'bold 15px sans-serif';
    ctx.textAlign = 'center';
    ctx.fillText(body.name, x, y - 28);
    S.mapSpaceMarkers.push({body, x, y, radius:(22 + index * 2) * Math.max(0.8, S.mapZoom * 0.9)});
  });

  if(S.space.mission){
    const targetMarker = S.mapSpaceMarkers.find(marker => marker.body.id === S.space.mission.bodyId);
    if(targetMarker){
      const progress = 1 - (S.space.mission.weeksLeft / S.space.mission.totalWeeks);
      const rocketX = lerp(earthPos.x, targetMarker.x, clamp(progress, 0, 1));
      const rocketY = lerp(earthPos.y, targetMarker.y, clamp(progress, 0, 1));
      ctx.setLineDash([10,6]);
      ctx.strokeStyle = 'rgba(79,195,247,.65)';
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.moveTo(earthPos.x, earthPos.y);
      ctx.lineTo(targetMarker.x, targetMarker.y);
      ctx.stroke();
      ctx.setLineDash([]);

      ctx.save();
      ctx.translate(rocketX, rocketY);
      ctx.rotate(Math.atan2(targetMarker.y - earthPos.y, targetMarker.x - earthPos.x));
      ctx.fillStyle = '#90caf9';
      ctx.beginPath();
      ctx.moveTo(18,0);
      ctx.lineTo(-10,-8);
      ctx.lineTo(-4,0);
      ctx.lineTo(-10,8);
      ctx.closePath();
      ctx.fill();
      ctx.fillStyle = '#ffb74d';
      ctx.beginPath();
      ctx.moveTo(-10,0);
      ctx.lineTo(-18,-5);
      ctx.lineTo(-18,5);
      ctx.closePath();
      ctx.fill();
      ctx.restore();
    }
  }

  ctx.fillStyle = 'rgba(255,255,255,.78)';
  ctx.font = '13px sans-serif';
  ctx.textAlign = 'left';
  ctx.fillText('宇宙マップ: 太陽と地球は別軌道。ロケットは地球から各天体へ向かいます', 20, 28);
}

// Map drag & zoom
const gameArea = document.getElementById('gameArea');
const rightPanelEl = document.getElementById('rightPanel');
let lastMapPointerDown = null;
gameArea.addEventListener('mouseenter', ()=> setMapFocus('map'));
rightPanelEl.addEventListener('mouseenter', ()=>{
  setMapFocus('panel');
  if(S.draggingWork && !isPlacementPanGesture()) cancelWorkPlacement();
  if(S.draggingRoute && !isPlacementPanGesture()) cancelRoutePlacement();
});
gameArea.addEventListener('contextmenu', e=>{
  e.preventDefault();
  if(S.currentMapMode !== 'japan') return;
  if(S.draggingWork || S.draggingRoute || S.draggingDisaster || S.draggingStrike) return;
  const pointer = lastMapPointerDown;
  const heldTooLong = pointer ? (performance.now() - pointer.time) > 420 : true;
  const movedTooFar = pointer ? pointer.moved : false;
  if(!pointer || pointer.button !== 2 || heldTooLong || movedTooFar) return;
  const rect = canvas.getBoundingClientRect();
  const mx = (e.clientX - rect.left) / rect.width * mapW;
  const my = (e.clientY - rect.top) / rect.height * mapH;
  const worldPos = canvasToWorld(mx, my);
  const prefectureMarker = getPrefectureMarkerAtCanvas(mx, my);
  const prefecture = prefectureMarker || getClosestPrefecture(worldPos.x, worldPos.y, S.mapZoom >= 1.3 ? 0.024 : 0.034);
  if(!prefecture) return;
  S.japanFocusPrefecture = prefecture.id;
  scheduleMapHudRefresh();
  scheduleMapDraw();
  openRegionalCompanyModal(prefecture.id);
});
gameArea.addEventListener('mousedown', e=>{
  lastMapPointerDown = {button:e.button, x:e.clientX, y:e.clientY, time:performance.now(), moved:false};
  if(S.currentMapMode === 'japan' && S.prefectureEnhancementBrush?.active && e.button === 1){
    e.preventDefault();
    const rect = canvas.getBoundingClientRect();
    const canvasX = (e.clientX - rect.left) / rect.width * mapW;
    const canvasY = (e.clientY - rect.top) / rect.height * mapH;
    S.prefectureBrushSelection = {startX:canvasX, startY:canvasY, endX:canvasX, endY:canvasY};
    S.mapDragging = false;
    drawMap();
    return;
  }
  if(S.currentMapMode === 'japan' && S.mapTraceActive && e.button === 0){
    e.preventDefault();
    const rect = canvas.getBoundingClientRect();
    const canvasX = (e.clientX - rect.left) / rect.width * mapW;
    const canvasY = (e.clientY - rect.top) / rect.height * mapH;
    const worldPos = canvasToWorld(canvasX, canvasY);
    S.mapTraceDrawing = true;
    S.mapTracePoints = [formatMapTracePoint(worldPos)];
    updateMapTracePanel();
    drawMap();
    return;
  }
  if((S.draggingWork || S.draggingRoute || S.draggingDisaster || S.draggingStrike) && e.button === 2){
    S.mapDragging = true;
    S.mapDragStart = {x: e.clientX, y: e.clientY, ox: S.mapOffsetX, oy: S.mapOffsetY};
    return;
  }
  if(S.draggingWork){
    if(e.button !== 0) return;
    const rect = canvas.getBoundingClientRect();
    const canvasX = (e.clientX - rect.left) / rect.width * mapW;
    const canvasY = (e.clientY - rect.top) / rect.height * mapH;
    const worldPos = canvasToWorld(canvasX, canvasY);
    S.draggingWork.x = worldPos.x;
    S.draggingWork.y = worldPos.y;
    drawMap();
    return;
  }
  if(S.draggingDisaster){
    if(e.button !== 0) return;
    const rect = canvas.getBoundingClientRect();
    const canvasX = (e.clientX - rect.left) / rect.width * mapW;
    const canvasY = (e.clientY - rect.top) / rect.height * mapH;
    const worldPos = canvasToWorld(canvasX, canvasY);
    S.draggingDisaster.x = worldPos.x;
    S.draggingDisaster.y = worldPos.y;
    drawMap();
    return;
  }
  if(S.draggingRoute){
    if(e.button !== 0) return;
    const rect = canvas.getBoundingClientRect();
    const canvasX = (e.clientX - rect.left) / rect.width * mapW;
    const canvasY = (e.clientY - rect.top) / rect.height * mapH;
    const worldPos = canvasToWorld(canvasX, canvasY);
    S.draggingRoute.x = worldPos.x;
    S.draggingRoute.y = worldPos.y;
    drawMap();
    return;
  }
  S.mapDragging = true;
  S.mapDragStart = {x: e.clientX, y: e.clientY, ox: S.mapOffsetX, oy: S.mapOffsetY};
});
gameArea.addEventListener('mousemove', e=>{
  const rect = canvas.getBoundingClientRect();
  const canvasX = (e.clientX - rect.left) / rect.width * mapW;
  const canvasY = (e.clientY - rect.top) / rect.height * mapH;
  S.mapCursorWorld = canvasToWorld(canvasX, canvasY);
  if(lastMapPointerDown && !lastMapPointerDown.moved){
    const dx = e.clientX - lastMapPointerDown.x;
    const dy = e.clientY - lastMapPointerDown.y;
    if(Math.hypot(dx, dy) > 7) lastMapPointerDown.moved = true;
  }
  if(S.mapTraceOpen) updateMapTracePanel();
  if(S.currentMapMode === 'japan' && S.prefectureBrushSelection){
    S.prefectureBrushSelection.endX = canvasX;
    S.prefectureBrushSelection.endY = canvasY;
    drawMap();
    return;
  }
  if(S.currentMapMode === 'japan' && S.mapTraceDrawing){
    const worldPos = canvasToWorld(canvasX, canvasY);
    const point = formatMapTracePoint(worldPos);
    const prev = (S.mapTracePoints || [])[S.mapTracePoints.length - 1];
    if(!prev || Math.hypot(prev.x - point.x, prev.y - point.y) >= 0.0032){
      S.mapTracePoints.push(point);
      if(S.mapTracePoints.length > 720) S.mapTracePoints.shift();
      updateMapTracePanel();
    }
    drawMap();
    return;
  }
  if(isPlacementPanGesture()){
    const viewport = getMapViewport();
    const dx = (((e.clientX - S.mapDragStart.x) / rect.width) * mapW) / Math.max(1, viewport.w * S.mapZoom);
    const dy = (((e.clientY - S.mapDragStart.y) / rect.height) * mapH) / Math.max(1, viewport.h * S.mapZoom);
    S.mapOffsetX = S.mapDragStart.ox + dx;
    S.mapOffsetY = S.mapDragStart.oy + dy;
    clampMapView();
    storeCurrentView();
    const worldAfterPan = canvasToWorld(canvasX, canvasY);
    if(S.draggingWork){
      S.draggingWork.x = worldAfterPan.x;
      S.draggingWork.y = worldAfterPan.y;
    } else if(S.draggingDisaster){
      S.draggingDisaster.x = worldAfterPan.x;
      S.draggingDisaster.y = worldAfterPan.y;
    } else if(S.draggingRoute){
      S.draggingRoute.x = worldAfterPan.x;
      S.draggingRoute.y = worldAfterPan.y;
    }
    drawMap();
    return;
  }
  if(S.draggingWork){
    const worldPos = canvasToWorld(canvasX, canvasY);
    S.draggingWork.x = worldPos.x;
    S.draggingWork.y = worldPos.y;
    drawMap();
    return;
  }
  if(S.draggingDisaster){
    const worldPos = canvasToWorld(canvasX, canvasY);
    S.draggingDisaster.x = worldPos.x;
    S.draggingDisaster.y = worldPos.y;
    drawMap();
    return;
  }
  if(S.draggingRoute){
    const worldPos = canvasToWorld(canvasX, canvasY);
    S.draggingRoute.x = worldPos.x;
    S.draggingRoute.y = worldPos.y;
    drawMap();
    return;
  }
  if(S.currentMapMode === 'world'){
    const hoveredCountry = getWorldCountryAtCanvas(canvasX, canvasY, 10);
    if(hoveredCountry){
      showTooltip(`${hoveredCountry.name} / ${formatDebugWorldCoords(hoveredCountry)}`, e.clientX, e.clientY);
    } else {
      const hoveredIncident = getHoveredMapIncidentAtCanvas(canvasX, canvasY, 'world');
      if(hoveredIncident) showTooltip(hoveredIncident.label, e.clientX, e.clientY);
      else hideTooltip();
    }
  } else if(S.currentMapMode === 'japan'){
    const prevHoverCompany = S.hoverCompanyName || '';
    const companyHoverEnabled = S.currentPanel !== 'infra';
    const hoveredCompanyEntry = companyHoverEnabled
      ? (S.mapCompanyMarkers || [])
        .map(marker => ({marker, dist:Math.hypot(canvasX - marker.x, canvasY - marker.y)}))
        .filter(entry => entry.dist <= entry.marker.radius + 5)
        .sort((a,b) => a.dist - b.dist)[0]
      : null;
    if(hoveredCompanyEntry){
      S.hoverCompanyName = hoveredCompanyEntry.marker.company.name;
      S.hoverPrefectureId = hoveredCompanyEntry.marker.company.prefecture || null;
      showTooltip(`${hoveredCompanyEntry.marker.company.name} / ${sectors[hoveredCompanyEntry.marker.company.sector]?.name || hoveredCompanyEntry.marker.company.sector}`, e.clientX, e.clientY);
    } else {
      S.hoverCompanyName = '';
      S.hoverPrefectureId = getPrefectureMarkerAtCanvas(canvasX, canvasY)?.id || null;
      const hoveredIncident = getHoveredMapIncidentAtCanvas(canvasX, canvasY, 'japan');
      if(hoveredIncident) showTooltip(hoveredIncident.label, e.clientX, e.clientY);
      else hideTooltip();
    }
    if(prevHoverCompany !== (S.hoverCompanyName || '')){
      drawMap();
      return;
    }
  } else {
    S.hoverCompanyName = '';
    S.hoverPrefectureId = null;
    hideTooltip();
  }
  if(!S.mapDragging) return;
  const viewport = getMapViewport();
  const dx = (((e.clientX - S.mapDragStart.x) / rect.width) * mapW) / Math.max(1, viewport.w * S.mapZoom);
  const dy = (((e.clientY - S.mapDragStart.y) / rect.height) * mapH) / Math.max(1, viewport.h * S.mapZoom);
  S.mapOffsetX = S.mapDragStart.ox + dx;
  S.mapOffsetY = S.mapDragStart.oy + dy;
  clampMapView();
  storeCurrentView();
  drawMap();
});
gameArea.addEventListener('mouseup', e=>{
  if(S.currentMapMode === 'japan' && e.button === 1 && S.prefectureBrushSelection){
    const selection = S.prefectureBrushSelection;
    S.prefectureBrushSelection = null;
    drawMap();
    if(window.applyPrefectureEnhancementSelection) window.applyPrefectureEnhancementSelection(selection);
    return;
  }
  if(S.currentMapMode === 'japan' && e.button === 0 && S.mapTraceDrawing){
    S.mapTraceDrawing = false;
    updateMapTracePanel();
    drawMap();
    return;
  }
  if(e.button === 2){
    S.mapDragging = false;
    return;
  }
  if(S.draggingWork){
    const rect = canvas.getBoundingClientRect();
    const canvasX = (e.clientX - rect.left) / rect.width * mapW;
    const canvasY = (e.clientY - rect.top) / rect.height * mapH;
    const worldPos = canvasToWorld(canvasX, canvasY);
    const keepType = S.draggingWork.type;
    const placed = placePublicWork(keepType, worldPos);
    if(placed && S.buildRepeatMode && S.money >= (publicWorksCatalog[keepType]?.cost || 0) && S.polCap >= (publicWorksCatalog[keepType]?.pol || 0)){
      S.draggingWork = {type:keepType, x:worldPos.x, y:worldPos.y};
    } else if(placed){
      S.draggingWork = null;
    }
    return;
  }
  if(S.draggingDisaster){
    return;
  }
  if(S.draggingRoute){
    S.mapDragging = false;
    return;
  }
  S.mapDragging = false;
});
gameArea.addEventListener('mouseleave', ()=> {
  S.mapDragging = false;
  S.mapTraceDrawing = false;
  S.hoverCompanyName = '';
  S.hoverPrefectureId = null;
  S.mapCursorWorld = null;
  lastMapPointerDown = null;
  hideTooltip();
  updateMapTracePanel();
});
gameArea.addEventListener('wheel', e=>{
  e.preventDefault();
  const rect = canvas.getBoundingClientRect();
  const canvasX = (e.clientX - rect.left) / rect.width * mapW;
  const canvasY = (e.clientY - rect.top) / rect.height * mapH;
  const worldBefore = canvasToWorld(canvasX, canvasY);
  const zoomStep = clamp(0.24 + Math.abs(e.deltaY) * 0.0004, 0.24, 0.62);
  const delta = e.deltaY > 0 ? -zoomStep : zoomStep;
  const bounds = getMapBounds(S.currentMapMode);
  const currentZoom = S.mapZoom;
  const nextZoom = Math.max(bounds.minZoom, Math.min(bounds.maxZoom, currentZoom + delta));
  if(Math.abs(nextZoom - currentZoom) < 0.0001) return;
  const viewport = getMapViewport();
  S.mapZoom = nextZoom;
  const canvasAfter = worldToCanvas(worldBefore.x, worldBefore.y);
  S.mapOffsetX += (canvasX - canvasAfter.x) / Math.max(1, viewport.w * nextZoom);
  S.mapOffsetY += (canvasY - canvasAfter.y) / Math.max(1, viewport.h * nextZoom);
  clampMapView();
  storeCurrentView();
  drawMap();
});

// Click on map to find region/company
gameArea.addEventListener('click', e=>{
  const wasRegularPan = !!(lastMapPointerDown && lastMapPointerDown.button === 0 && lastMapPointerDown.moved && !(S.draggingWork || S.draggingRoute || S.draggingDisaster || S.draggingStrike));
  if(wasRegularPan) return;
  if(S.currentMapMode === 'japan' && S.mapTraceActive) return;
  const rect = canvas.getBoundingClientRect();
  const mx = (e.clientX - rect.left) / rect.width * mapW;
  const my = (e.clientY - rect.top) / rect.height * mapH;

  if(S.currentMapMode === 'space'){
    const target = (S.mapSpaceMarkers || []).find(marker => Math.hypot(mx - marker.x, my - marker.y) <= marker.radius + 6);
    if(target){
      S.spaceFocusBody = target.body.id;
      renderPanel();
      drawMap();
    }
    return;
  }

  if(S.currentMapMode === 'world'){
    const target = getWorldCountryAtCanvas(mx, my, 12);
    if(target){
      if(S.draggingDisaster && S.draggingDisaster.target === 'world'){
        applyChaosToCountry(S.draggingDisaster.type, target.id);
        updateMapHud();
        drawMap();
        return;
      }
      if(S.draggingStrike){
        fireNuclearStrike(target.id);
        updateMapHud();
        drawMap();
        return;
      }
      S.worldFocusCountry = target.id;
      renderPanel();
      updateMapHud();
      drawMap();
      openWorldCountryModal(target.id);
    }
    return;
  }

  const worldPos = canvasToWorld(mx, my);
  if(S.draggingDisaster && S.draggingDisaster.target === 'japan'){
    applyChaosAtPoint(S.draggingDisaster.type, worldPos);
    updateMapHud();
    drawMap();
    return;
  }
  const incidentDetail = getIncidentDetailAtCanvas(mx, my, 'japan');
  if(incidentDetail){
    openIncidentDetailModal(incidentDetail);
    return;
  }

  const prefectureMarker = getPrefectureMarkerAtCanvas(mx, my);
  const routeThreshold = S.mapZoom >= 1.3 ? 0.052 : 0.074;
  const prefectureThreshold = S.mapZoom >= 1.3 ? 0.024 : 0.034;
  const routeAnchor = S.draggingRoute ? getClosestTransportAnchor(worldPos.x, worldPos.y, routeThreshold) : null;
  const prefecture = S.draggingRoute
    ? resolveTransportEndpointPrefecture(mx, my, worldPos)
    : (routeAnchor ? prefectureMap[routeAnchor.prefectureId] : (prefectureMarker || getClosestPrefecture(worldPos.x, worldPos.y, prefectureThreshold)));
  if(S.draggingRoute){
    if(!prefecture){
      addEvent('❌ 路線の端点は都道府県を選んでください', 'bad');
      return;
    }
    if(!S.draggingRoute.fromPrefectureId){
      S.draggingRoute.fromPrefectureId = prefecture.id;
      const anchor = getPrefectureTransitAnchor(prefecture.id);
      if(anchor){
        S.draggingRoute.x = anchor.x;
        S.draggingRoute.y = anchor.y;
      }
      S.japanFocusPrefecture = prefecture.id;
      updateMapHud();
      drawMap();
      return;
    }
    const keepType = S.draggingRoute.type;
    const keepChainId = S.draggingRoute.chainId;
    const nextSegmentIndex = (S.draggingRoute.segmentIndex || 0) + 1;
    const placed = placeTransportRoute(S.draggingRoute.fromPrefectureId, prefecture.id, keepType);
    if(placed && S.transportRepeatMode && S.money >= (transportNetworkCatalog[keepType]?.cost || 0) && S.polCap >= (transportNetworkCatalog[keepType]?.pol || 0)){
      const anchor = getPrefectureTransitAnchor(prefecture.id) || prefecture;
      S.draggingRoute = {type:keepType, fromPrefectureId:prefecture.id, x:anchor.x, y:anchor.y, chainId:keepChainId, segmentIndex:nextSegmentIndex};
    } else if(placed){
      S.draggingRoute = null;
    }
    updateMapHud();
    drawMap();
    return;
  }
  if(prefecture){
    S.japanFocusPrefecture = prefecture.id;
    updateMapHud();
    drawMap();
    openPrefectureModal(prefecture.id);
    return;
  }

  if(S.currentPanel !== 'infra'){
    const companyMarker = (S.mapCompanyMarkers || [])
      .map(marker => ({marker, dist:Math.hypot(mx - marker.x, my - marker.y)}))
      .filter(entry => entry.dist <= entry.marker.radius + 6)
      .sort((a,b) => a.dist - b.dist)[0];
    if(companyMarker){
      openCompanyDetail(companyMarker.marker.company.name);
      return;
    }
  }

  // Find clicked region
  let closest = null, closestDist = Infinity;
  Object.entries(regions).forEach(([k,r])=>{
    const d = Math.hypot(worldPos.x - r.cx, worldPos.y - r.cy);
    if(d < closestDist){ closestDist = d; closest = k; }
  });
  if(closestDist < 0.08) openRegionInfoModal(closest);
});

function renderPrefectureModal(prefectureId, mode='prefecture'){
  const prefecture = prefectureMap[prefectureId];
  if(!prefecture) return '';
  const companiesInPrefecture = getPrefectureCompanies(prefectureId);
  const regionCompanies = companies.filter(company => company.region === prefecture.region);
  const region = regions[prefecture.region];
  const activeDisaster = region?.disaster && region.disaster.weeksLeft > 0 ? region.disaster : null;
  const showPrefecture = mode !== 'region';
  const topCompany = companiesInPrefecture[0] || regionCompanies[0] || null;
  return `<div class="section">
    <div class="prefecture-hero">
      <div class="title">${prefecture.name}</div>
      <div class="sub">${region?.name || prefecture.region} / 県内企業 ${companiesInPrefecture.length}社 / 地方企業 ${regionCompanies.length}社${topCompany ? `<br>主な企業: ${topCompany.name}` : ''}</div>
    </div>
    <div class="save-actions" style="margin-bottom:10px">
      <button class="btn btn-sm ${showPrefecture ? 'btn-blue active' : ''}" onclick="switchPrefectureModalView('${prefectureId}','prefecture')">県情報</button>
      <button class="btn btn-sm ${!showPrefecture ? 'btn-green active' : ''}" onclick="switchPrefectureModalView('${prefectureId}','region')">${region?.name || prefecture.region}企業</button>
    </div>
    ${showPrefecture ? `
      <div class="detail-grid">
        <div class="dg-item"><div class="dg-label">地方</div><div class="dg-value">${region?.name || prefecture.region}</div></div>
        <div class="dg-item"><div class="dg-label">県内企業数</div><div class="dg-value">${companiesInPrefecture.length}</div></div>
        <div class="dg-item"><div class="dg-label">地方企業数</div><div class="dg-value">${regionCompanies.length}</div></div>
        <div class="dg-item"><div class="dg-label">開発度</div><div class="dg-value">${region?.developed || 0}</div></div>
        <div class="dg-item"><div class="dg-label">主資源</div><div class="dg-value">${(region?.resources || []).slice(0, 2).map(key => resources[key]?.name || key).join(' / ') || 'なし'}</div></div>
        <div class="dg-item"><div class="dg-label">注目操作</div><div class="dg-value">右クリックで地方企業</div></div>
        <div class="dg-item"><div class="dg-label">災害状況</div><div class="dg-value">${activeDisaster ? `${activeDisaster.type} / 残り${activeDisaster.weeksLeft}週` : '平常'}</div></div>
      </div>
      <div class="compact-note" style="margin-top:8px">${prefecture.name} の企業一覧です。左クリックは県情報、右クリックは地方企業を開きます。</div>
      ${activeDisaster ? `<div class="headline-card bad" style="margin-top:8px"><div class="meta">自然災害継続中</div><div class="body">${activeDisaster.type} が継続中です。生産・売上・地域安定へ継続ダメージが入っています。残り ${activeDisaster.weeksLeft}週。</div></div>` : ''}
      ${companiesInPrefecture.map(company => {
        const sector = sectors[company.sector];
        return `<div class="company" style="margin-top:8px;border-left-color:${sector?.color || '#90caf9'}" onclick="openCompanyDetail('${company.name}')">
          <div class="c-left"><span class="c-name">${sector?.icon || '🏢'}${company.name}</span></div>
          <div class="c-right"><span class="c-price">${formatMoneyCompact(company.price)}</span></div>
        </div>`;
      }).join('') || `<div class="compact-note" style="margin-top:8px">この県に登録企業はまだありません。</div>`}
    ` : `
      <div class="compact-note" style="margin-bottom:8px">${prefecture.name} が属する ${region?.name || prefecture.region} の企業一覧です。</div>
      ${regionCompanies.map(company => {
        const sector = sectors[company.sector];
        const inPrefecture = company.prefecture === prefectureId;
        return `<div class="company" style="margin-top:8px;border-left-color:${sector?.color || '#90caf9'}" onclick="openCompanyDetail('${company.name}')">
          <div class="c-left" style="display:flex;flex-direction:column;align-items:flex-start">
            <span class="c-name">${sector?.icon || '🏢'}${company.name}</span>
            <span class="compact-note">${prefectureMap[company.prefecture]?.name || company.prefecture}${inPrefecture ? ' / 選択中の県' : ''}</span>
          </div>
          <div class="c-right"><span class="c-price">${formatMoneyCompact(company.price)}</span></div>
        </div>`;
      }).join('')}
    `}
  </div>`;
}

window.switchPrefectureModalView = function(prefectureId, mode){
  const prefecture = prefectureMap[prefectureId];
  if(!prefecture) return;
  document.getElementById('modalTitle').textContent = `🗾 ${prefecture.name}`;
  document.getElementById('modalContent').innerHTML = renderPrefectureModal(prefectureId, mode);
};

function openPrefectureModal(prefectureId, mode='prefecture'){
  const prefecture = prefectureMap[prefectureId];
  if(!prefecture) return;
  const modal = document.getElementById('actionModal');
  const modalCard = modal.querySelector('.modal');
  if(modalCard) modalCard.classList.remove('modal-world-country','modal-cyber','modal-finance','modal-company');
  if(modalCard) modalCard.classList.add('modal-prefecture');
  S.japanFocusPrefecture = prefectureId;
  document.getElementById('modalTitle').textContent = `🗾 ${prefecture.name}`;
  document.getElementById('modalContent').innerHTML = renderPrefectureModal(prefectureId, mode);
  modal.classList.add('show');
}

function openRegionInfoModal(rk){
  const r = regions[rk];
  const modal = document.getElementById('actionModal');
  document.getElementById('modalTitle').textContent = `🗺 ${r.name}`;
  let html = `<div class="detail-grid">
    <div class="dg-item"><div class="dg-label">開発レベル</div><div class="dg-value">${r.developed}/${r.devSlots}</div></div>
    <div class="dg-item"><div class="dg-label">タイプ</div><div class="dg-value">${r.isSea?'海洋':'陸地'}</div></div>
  </div><p style="font-size:10px;margin-top:6px">資源: ${r.resources.map(s=>resources[s]?.name||s).join(', ')}</p>`;
  const discovered = S.discoveredMaterials.filter(d => d.region === rk);
  if(discovered.length > 0){
    html += `<p style="color:#f39c12;font-size:10px;margin-top:4px">✦ 発見済み特殊素材: ${discovered.map(d=>{
      const m = [...uniqueMaterials, ...spaceMaterials].find(u=>u.id===d.id); return m?m.name:'?';
    }).join(', ')}</p>`;
  }
  document.getElementById('modalContent').innerHTML = html;
  modal.classList.add('show');
}

function openCompanyListModal(rk, comps, options={}){
  const r = regions[rk];
  const modal = document.getElementById('actionModal');
  document.getElementById('modalTitle').textContent = options.title || `🗺 ${r.name} の企業`;
  let html = options.note ? `<div class="compact-note" style="margin-bottom:8px">${options.note}</div>` : '';
  comps.forEach(c => {
    const sec = sectors[c.sector];
    html += `<div style="padding:6px;margin:4px 0;background:rgba(0,0,0,.2);border-radius:4px;border-left:3px solid ${sec.color};cursor:pointer"
      onclick="openCompanyDetail('${c.name}')">
      <div style="font-weight:bold">${sec.icon} ${c.name} <span style="color:#888;font-size:9px">${sec.name}</span></div>
      <div style="font-size:10px">${formatMoneyCompact(c.price)} <span style="font-size:9px;color:#888">${c.desc}</span></div>
    </div>`;
  });
  document.getElementById('modalContent').innerHTML = html;
  modal.classList.add('show');
}

function openRegionalCompanyModal(prefectureId){
  const prefecture = prefectureMap[prefectureId];
  if(!prefecture) return;
  openPrefectureModal(prefectureId, 'region');
}

window.focusWorldCountry = function(countryId, options={}){
  const country = getWorldCountry(countryId);
  if(!country) return;
  S.worldFocusCountry = country.id;
  if(options.switchMap){
    storeCurrentView();
    S.currentMapMode = 'world';
    loadViewForMode('world');
  }
  if(options.switchPanel) S.currentPanel = 'world';
  renderPanel();
  updateMapHud();
  drawMap();
  if(options.openModal) openWorldCountryModal(country.id);
};

function formatWarUnitsCompact(units){
  return `歩 ${units.infantry || 0} / 戦 ${units.tanks || 0} / 航 ${units.fighters || 0} / 艦 ${units.ships || 0} / 無 ${units.drones || 0}`;
}

function renderWarDispatchControls(countryId, mode='modal'){
  if(!S.activeWar || S.activeWar.countryId !== countryId) return '';
  const war = S.activeWar;
  const partner = tradePartners[war.partnerIdx];
  const compact = mode === 'modal';
  const refreshAction = mode === 'battle' ? `openWarBattleCommand()` : mode === 'panel' ? `renderPanel()` : `openWorldCountryModal('${countryId}')`;
  return `<div class="section" style="margin-top:10px">
    <h3>⚔ 送軍指揮</h3>
    <div class="compact-note">前線輸送 ${war.travelWeeks}週 / 日本前線 ${formatWarUnitsCompact(war.jpFront)} / 敵前線 ${canViewWarIntel(countryId) ? formatWarUnitsCompact(war.enemyFront) : '不明'}</div>
    <div class="battle-reinforcement-grid" style="margin-top:10px">
      ${getUnitKeys().map(key => {
        const unit = getMilitaryCatalog()[key];
        const available = S.militaryInventory[key] || 0;
        return `<div class="battle-reinforcement-card">
          <div class="top"><span>${unit.icon} ${unit.name}</span><span>予備 ${available}</span></div>
          <div class="save-actions">
            <button class="btn btn-sm btn-blue" onclick="sendWarUnits('${key}',25); ${refreshAction}">x25</button>
            <button class="btn btn-sm" onclick="sendWarUnits('${key}',100); ${refreshAction}">x100</button>
            <button class="btn btn-sm" onclick="sendWarUnits('${key}',500); ${refreshAction}">x500</button>
            <button class="btn btn-sm btn-purple" onclick="sendWarUnits('${key}','all'); ${refreshAction}">全部</button>
          </div>
        </div>`;
      }).join('')}
    </div>
    <div class="battle-actions" style="margin-top:10px">
      ${mode === 'battle' ? '' : `<button class="btn btn-yellow" onclick="openWarBattleCommand()">${compact ? '会戦画面' : '⚔ 会戦画面を開く'}</button>`}
      <button class="btn" onclick="returnWarUnits(); ${refreshAction}">前線部隊を撤退</button>
      <button class="btn btn-blue" onclick="focusWorldCountry('${countryId}', {switchMap:true, switchPanel:true}); closeModal();">世界マップで見る</button>
    </div>
    <div class="compact-note" style="margin-top:8px">敵予備: ${canViewWarIntel(countryId) ? formatWarUnitsCompact(war.enemyReserves) : `${partner?.name || '敵国'} の情報は限定的`} / 輸送中 日本 ${war.jpDispatches.length}便・敵 ${war.enemyDispatches.length}便</div>
  </div>`;
}

window.openWarBattleCommand = function(auto=false){
  const war = S.activeWar;
  if(!war) return;
  const country = getWorldCountry(war.countryId);
  const partner = tradePartners[war.partnerIdx];
  const jpFrontPower = Math.round(getUnitsPower(war.jpFront, 'jp') + S.defPower * 0.25 + S.cyberPower * 0.08);
  const enemyFrontPower = Math.round(getUnitsPower(war.enemyFront, 'enemy', partner) + partner.nukePower * 3);
  const modal = document.getElementById('actionModal');
  document.getElementById('modalTitle').textContent = auto ? `⚔ 会戦発生: ${country.name}` : `⚔ 会戦画面: ${country.name}`;
  document.getElementById('modalContent').innerHTML = `<div class="battle-screen">
    <div class="battle-hero">
      <div class="title">${country.name} 戦線</div>
      <div class="subtitle">部隊が到着し、前線で会戦可能な状態です。ここで戦力差を見て、そのまま戦うか、増援を待つか、撤退するかを決められます。</div>
    </div>
    <div class="battle-vs-grid">
      <div class="battle-side jp">
        <div class="label">日本軍 前線戦力</div>
        <div class="power">${jpFrontPower.toLocaleString()}</div>
        <div class="meta">${formatWarUnitsCompact(war.jpFront)}<br>輸送中 ${war.jpDispatches.length}便 / 本土予備 ${formatWarUnitsCompact(S.militaryInventory)}</div>
      </div>
      <div class="battle-versus">VS</div>
      <div class="battle-side enemy">
        <div class="label">${partner.name} 軍 前線戦力</div>
        <div class="power">${enemyFrontPower.toLocaleString()}</div>
        <div class="meta">${canViewWarIntel(country.id) ? `${formatWarUnitsCompact(war.enemyFront)}<br>輸送中 ${war.enemyDispatches.length}便 / 敵予備 ${formatWarUnitsCompact(war.enemyReserves)}` : '敵前線情報は一部のみ判明'}</div>
      </div>
    </div>
    <div class="macro-grid">
      <div class="macro-card"><div class="macro-label">戦況スコア</div><div class="macro-value">${war.warScore.toFixed(1)}</div><div class="compact-note">正なら日本軍優勢、負なら敵軍優勢です。</div></div>
      <div class="macro-card"><div class="macro-label">輸送距離</div><div class="macro-value">${war.travelWeeks}週</div><div class="compact-note">海運・通信・宇宙中継の影響を受けます。</div></div>
      <div class="macro-card"><div class="macro-label">勝利報酬候補</div><div class="macro-value">${formatMoneyCompact(Math.round(2800 + partner.gdpUsd * 600 + partner.economy * 40))}</div><div class="compact-note">投資権解禁と戦利金が得られます。</div></div>
      <div class="macro-card"><div class="macro-label">敗北リスク</div><div class="macro-value">${Math.max(1, Math.round(Math.abs(enemyFrontPower - jpFrontPower) / Math.max(1000, jpFrontPower) * 100))}%</div><div class="compact-note">劣勢なら撤退や増援も選択肢です。</div></div>
    </div>
    ${renderWarDispatchControls(country.id, 'battle')}
    <div class="battle-actions">
      <button class="btn btn-yellow" onclick="fightWarBattle(); openWarBattleCommand();">たたかう</button>
      <button class="btn" onclick="returnWarUnits(); openWarBattleCommand();">撤退する</button>
      <button class="btn btn-blue" onclick="focusWorldCountry('${country.id}', {switchMap:true, switchPanel:true}); closeModal();">世界マップへ戻る</button>
    </div>
    <div class="battle-log">
      ${(war.logs || []).map(entry => `<div class="advisor-item ${entry.type === 'bad' ? 'danger' : entry.type === 'good' ? '' : 'warn'}">${entry.week} ${entry.text}</div>`).join('') || '<div class="compact-note">まだ戦況ログはありません。</div>'}
    </div>
  </div>`;
  modal.classList.add('show');
};

function openWarVictoryModal(country, reward){
  const modal = document.getElementById('actionModal');
  document.getElementById('modalTitle').textContent = `🏆 ${country.name} 戦勝結果`;
  document.getElementById('modalContent').innerHTML = `<div class="battle-screen">
    <div class="battle-hero">
      <div class="title">${country.name} に勝利</div>
      <div class="subtitle">投資権を獲得し、戦利金や戦略効果が反映されました。ここから海外株、外交、追加攻勢へつなげられます。</div>
    </div>
    <div class="macro-grid">
      <div class="macro-card"><div class="macro-label">戦利金</div><div class="macro-value">${formatMoneyCompact(reward.spoils)}</div></div>
      <div class="macro-card"><div class="macro-label">投資権</div><div class="macro-value">${reward.investment ? '解禁' : '未解禁'}</div></div>
      <div class="macro-card"><div class="macro-label">相手GDP損害</div><div class="macro-value">-${reward.gdpDamage.toFixed(1)}%</div></div>
      <div class="macro-card"><div class="macro-label">相手人口損害</div><div class="macro-value">-${reward.populationLoss.toFixed(1)}百万人</div></div>
      <div class="macro-card"><div class="macro-label">相手安定度</div><div class="macro-value">-${reward.stabilityDamage.toFixed(1)}</div></div>
      <div class="macro-card"><div class="macro-label">友好度下限</div><div class="macro-value">${reward.relationFloor}</div></div>
    </div>
    <div class="battle-actions">
      <button class="btn btn-purple" onclick="S.currentPanel='foreign'; renderPanel(); closeModal();">海外株を見る</button>
      <button class="btn btn-blue" onclick="focusWorldCountry('${country.id}', {switchMap:true, switchPanel:true}); closeModal();">世界マップで確認</button>
      <button class="btn" onclick="closeModal()">閉じる</button>
    </div>
  </div>`;
  modal.classList.add('show');
  showCinematicEvent({
    tone:'victory',
    badge:'VICTORY',
    icon:'🏆',
    title:`${country.name} に勝利`,
    subtitle:'中央突破に成功。戦利金と投資権を確保し、敵国の国家基盤へ大打撃を与えました。',
    duration:4600,
  });
}

function openWarDefeatModal(country, partner, reason=''){
  const modal = document.getElementById('actionModal');
  document.getElementById('modalTitle').textContent = `⚠ ${country.name} 戦線喪失`;
  document.getElementById('modalContent').innerHTML = `<div class="battle-screen">
    <div class="battle-hero">
      <div class="title">${country.name} との戦争に敗北</div>
      <div class="subtitle">${reason || '自軍の前線維持に失敗し、作戦を継続できなくなりました。'} 予備戦力を立て直してから再挑戦できます。</div>
    </div>
    <div class="macro-grid">
      <div class="macro-card"><div class="macro-label">敵国</div><div class="macro-value">${country.name}</div></div>
      <div class="macro-card"><div class="macro-label">敵友好度</div><div class="macro-value">${partner.relation.toFixed(0)}</div></div>
      <div class="macro-card"><div class="macro-label">支持率影響</div><div class="macro-value">-6</div></div>
      <div class="macro-card"><div class="macro-label">安定度影響</div><div class="macro-value">-6</div></div>
    </div>
    <div class="battle-actions">
      <button class="btn btn-blue" onclick="focusWorldCountry('${country.id}', {switchMap:true, switchPanel:true}); closeModal();">世界マップへ戻る</button>
      <button class="btn" onclick="closeModal()">閉じる</button>
    </div>
  </div>`;
  modal.classList.add('show');
  showCinematicEvent({
    tone:'defeat',
    badge:'RETREAT',
    icon:'⚠',
    title:`${country.name} 戦線喪失`,
    subtitle:reason || '前線の維持に失敗し、日本軍は再編成へ移りました。',
    duration:3600,
  });
}

window.openWorldCountryModal = function(countryId){
  const country = getWorldCountry(countryId);
  const partner = getWorldCountryPartner(country);
  const snapshot = getWorldCountrySnapshot(country);
  const intel = getDisplayMilitaryIntel(country.id);
  const spyStatus = getSpyIntelStatus(country.id);
  const modal = document.getElementById('actionModal');
  const modalCard = modal.querySelector('.modal');
  if(modalCard){
    modalCard.classList.remove('modal-cyber','modal-company','modal-prefecture','modal-finance');
    modalCard.classList.add('modal-world-country');
  }
  modal.dataset.action = 'world-country';
  document.getElementById('modalTitle').textContent = `🌐 ${country.name}`;
  document.getElementById('modalContent').innerHTML = `<div class="world-country-sheet">
      <div class="world-country-hero">
        <div class="title">${country.name}</div>
        <div class="sub">${country.note}<br>${partner ? `${country.name} と日本の友好度は ${Math.round(snapshot.relation)} です。` : '日本本体の基礎情報です。自国戦力は常時確認できます。'}</div>
      </div>
      <div class="section">
        <div class="detail-grid">
          <div class="dg-item"><div class="dg-label">GDP</div><div class="dg-value">${snapshot.gdp ? `${snapshot.gdp.toFixed ? snapshot.gdp.toFixed(0) : snapshot.gdp}兆` : '戦略指標'}</div></div>
          <div class="dg-item"><div class="dg-label">友好度</div><div class="dg-value">${Math.round(snapshot.relation)}</div></div>
          <div class="dg-item"><div class="dg-label">サイバー</div><div class="dg-value">${Number(snapshot.cyber).toFixed(0)}</div></div>
          <div class="dg-item"><div class="dg-label">核戦力</div><div class="dg-value">${snapshot.nuke}</div></div>
          <div class="dg-item"><div class="dg-label">防御戦力</div><div class="dg-value">${intel ? Math.round(intel.power) : '不明'}</div></div>
          <div class="dg-item"><div class="dg-label">前線情報</div><div class="dg-value">${intel ? `歩 ${intel.units.infantry} / 戦 ${intel.units.tanks}` : '友好か交戦で開示'}</div></div>
          <div class="dg-item"><div class="dg-label">人口</div><div class="dg-value">${partner?.population ? `${Math.round(partner.population)}百万人` : '-'}</div></div>
          <div class="dg-item"><div class="dg-label">名目GDP(実データ)</div><div class="dg-value">${snapshot.actualGdp ? `${snapshot.actualGdp.toFixed(2)}兆ドル` : '-'}</div></div>
          <div class="dg-item"><div class="dg-label">通貨</div><div class="dg-value">${snapshot.currencyCode || 'JPY'} / ${snapshot.currencyName || '円'}</div></div>
          <div class="dg-item"><div class="dg-label">インフレ</div><div class="dg-value">${Number(snapshot.inflation || 0).toFixed(1)}%</div></div>
          <div class="dg-item"><div class="dg-label">失業率</div><div class="dg-value">${Number(snapshot.unemployment || 0).toFixed(1)}%</div></div>
          <div class="dg-item"><div class="dg-label">戦災後遺症</div><div class="dg-value">${Number(snapshot.nuclearScars || 0).toFixed(1)}</div></div>
          <div class="dg-item"><div class="dg-label">諜報網</div><div class="dg-value">${spyStatus ? `${spyStatus.weeksLeft}週 / 潜入${spyStatus.activeMissions}班` : '-'}</div></div>
          <div class="dg-item"><div class="dg-label">調整用座標</div><div class="dg-value">${formatDebugWorldCoords(country)}</div></div>
        </div>
        <div class="world-sector-list">${country.displaySectors.map(sectorId => {
          const sector = sectors[sectorId];
          return sector ? `<span class="world-sector-pill">${sector.icon}${sector.name}</span>` : '';
        }).join('')}</div>
        ${partner ? `<div class="compact-note" style="margin-top:8px">輸出候補: ${partner.sells.map(item => resources[item]?.name || productCatalog[item]?.name || item).join(' / ')}<br>需要候補: ${partner.buys.map(item => resources[item]?.name || productCatalog[item]?.name || item).join(' / ')}</div>` : '<div class="compact-note" style="margin-top:8px">日本の視点では、ここが全ての交易・外交・投資の中心ハブです。</div>'}
        <div class="compact-note" style="margin-top:8px">主要企業: ${country.majorCompanies.join(' / ')}</div>
        <div class="world-country-actions">
          <button class="btn btn-sm btn-blue" onclick="focusWorldCountry('${country.id}', {switchMap:true, switchPanel:true}); closeModal();">マップで注目</button>
          ${partner ? `<button class="btn btn-sm btn-green" onclick="improveDiplomacy(${country.partnerIdx}); openWorldCountryModal('${country.id}')">外交改善</button>` : ''}
          ${partner ? `<button class="btn btn-sm" onclick="worsenDiplomacy(${country.partnerIdx}); openWorldCountryModal('${country.id}')">敵対工作</button>` : ''}
          ${partner ? `<button class="btn btn-sm btn-yellow" onclick="declareWar('${country.id}'); openWorldCountryModal('${country.id}')">宣戦</button>` : ''}
          ${partner ? `<button class="btn btn-sm" onclick="S.currentPanel='trade'; renderPanel(); closeModal();">貿易タブへ</button>` : ''}
          ${canInvestInCountry(country.id) ? `<button class="btn btn-sm btn-purple" onclick="S.currentPanel='foreign'; renderPanel(); closeModal();">海外株タブへ</button>` : ''}
        </div>
        ${partner ? `<div class="section" style="margin-top:10px">
          <h3>🕵 諜報潜入</h3>
          <div class="compact-note">保有諜報班 ${S.militaryInventory.spies || 0} / 成功すると軍事情報を一定期間奪取し、資金も持ち帰ります。露見すると損失と外交悪化が発生します。</div>
          <div class="save-actions" style="margin-top:8px">
            <button class="btn btn-sm btn-blue" onclick="sendSpyMission('${country.id}',1); openWorldCountryModal('${country.id}')">x1</button>
            <button class="btn btn-sm" onclick="sendSpyMission('${country.id}',5); openWorldCountryModal('${country.id}')">x5</button>
            <button class="btn btn-sm" onclick="sendSpyMission('${country.id}',10); openWorldCountryModal('${country.id}')">x10</button>
            <button class="btn btn-sm btn-purple" onclick="sendSpyMission('${country.id}','all'); openWorldCountryModal('${country.id}')">全部</button>
          </div>
          <div class="compact-note" style="margin-top:8px">${spyStatus ? `情報維持 ${spyStatus.weeksLeft}週 / 警戒度 ${spyStatus.alert.toFixed(1)} / 潜入中 ${spyStatus.activeMissions}` : ''}</div>
        </div>` : ''}
        ${canInvestInCountry(country.id) ? `<div class="status-badge good" style="margin-top:8px">戦勝済み: この国の企業に投資可能</div>` : partner ? `<div class="status-badge ${snapshot.relation <= 35 ? 'warn' : 'bad'}" style="margin-top:8px">投資条件: 戦争に勝利すると解禁</div>` : ''}
        ${renderWarDispatchControls(country.id, 'modal')}
      </div></div>`;
  modal.classList.add('show');
};

window.buyForeignStock = function(companyId, qty){
  ensureForeignCompanies();
  const company = S.foreignCompanies.find(entry => entry.id === companyId);
  if(!company) return;
  if(!canInvestInCountry(company.countryId)){ addEvent('❌ その国ではまだ投資権を得ていません', 'bad'); return; }
  const owned = S.foreignOwnedStocks[companyId] || 0;
  qty = Math.min(qty, Math.max(0, company.sharesOutstanding - owned));
  if(qty <= 0){ addEvent('ℹ これ以上は取得できません', ''); return; }
  const cost = company.price * qty;
  if(S.money < cost){ addEvent('❌ 資金不足', 'bad'); return; }
  S.money -= cost;
  S.foreignOwnedStocks[companyId] = owned + qty;
  addEvent(`🌍 ${company.name} を ${qty}株取得`, 'good');
  updateUI();
  openForeignCompanyDetail(companyId);
};

window.sellForeignStock = function(companyId, qty){
  ensureForeignCompanies();
  const company = S.foreignCompanies.find(entry => entry.id === companyId);
  if(!company) return;
  const owned = S.foreignOwnedStocks[companyId] || 0;
  qty = Math.min(qty, owned);
  if(qty <= 0) return;
  S.money += company.price * qty;
  S.foreignOwnedStocks[companyId] = owned - qty;
  if(S.foreignOwnedStocks[companyId] <= 0) delete S.foreignOwnedStocks[companyId];
  addEvent(`🌍 ${company.name} を ${qty}株売却`, 'good');
  updateUI();
  openForeignCompanyDetail(companyId);
};

window.buyForeignStockAll = function(companyId){
  ensureForeignCompanies();
  const company = S.foreignCompanies.find(entry => entry.id === companyId);
  if(!company) return;
  const owned = S.foreignOwnedStocks[companyId] || 0;
  const qty = Math.max(0, Math.min(company.sharesOutstanding - owned, Math.floor(S.money / Math.max(1, company.price))));
  if(qty <= 0){ addEvent('❌ 一括購入できる資金か残株がありません', 'bad'); return; }
  buyForeignStock(companyId, qty);
};

window.openForeignCompanyDetail = function(companyId){
  ensureForeignCompanies();
  const company = S.foreignCompanies.find(entry => entry.id === companyId);
  if(!company) return;
  const owned = S.foreignOwnedStocks[companyId] || 0;
  const ratio = clamp(owned / Math.max(1, company.sharesOutstanding) * 100, 0, 100);
  const modal = document.getElementById('actionModal');
  const modalCard = modal.querySelector('.modal');
  if(modalCard) modalCard.classList.remove('modal-world-country','modal-cyber','modal-finance','modal-prefecture');
  if(modalCard) modalCard.classList.add('modal-company');
  document.getElementById('modalTitle').textContent = `🌍 ${company.name}`;
  document.getElementById('modalContent').innerHTML = `<div class="section">
    <div class="detail-grid">
      <div class="dg-item"><div class="dg-label">所在国</div><div class="dg-value">${company.countryName}</div></div>
      <div class="dg-item"><div class="dg-label">セクター</div><div class="dg-value">${sectors[company.sector]?.name || company.sector}</div></div>
      <div class="dg-item"><div class="dg-label">株価</div><div class="dg-value">¥${Math.round(company.price).toLocaleString()}</div></div>
      <div class="dg-item"><div class="dg-label">保有率</div><div class="dg-value">${ratio.toFixed(2)}%</div></div>
      <div class="dg-item"><div class="dg-label">年間売上</div><div class="dg-value">¥${Math.round(company.revenue).toLocaleString()}</div></div>
      <div class="dg-item"><div class="dg-label">利益率</div><div class="dg-value">${(company.margin * 100).toFixed(1)}%</div></div>
    </div>
    <p class="compact-note" style="margin-top:8px">${company.desc}</p>
    <canvas id="foreignChart" width="560" height="180" style="width:100%;height:180px;background:rgba(0,0,0,.2);border-radius:8px;margin-top:8px"></canvas>
    <div class="save-actions" style="margin-top:8px">
      ${[1,10,50,100,250,500,1000].map(qty => `<button class="btn btn-sm btn-green" onclick="buyForeignStock('${company.id}', ${qty})">${qty}株購入</button>`).join('')}
      <button class="btn btn-sm btn-green" onclick="buyForeignStockAll('${company.id}')">全部買う</button>
      <button class="btn btn-sm" onclick="sellForeignStock('${company.id}', 1)" ${owned < 1 ? 'disabled' : ''}>1株売却</button>
      <button class="btn btn-sm" onclick="sellForeignStock('${company.id}', 50)" ${owned < 50 ? 'disabled' : ''}>50株売却</button>
      <button class="btn btn-sm" onclick="sellForeignStock('${company.id}', ${owned})" ${owned < 1 ? 'disabled' : ''}>全売却</button>
    </div>
  </div>`;
  modal.classList.add('show');
  setTimeout(() => {
    const chart = document.getElementById('foreignChart');
    if(!chart) return;
    const ctx2 = chart.getContext('2d');
    const data = (company.hist.length >= 2 ? company.hist : [company.price, company.price]).slice(-260);
    const min = Math.min(...data);
    const max = Math.max(...data);
    const range = max - min || 1;
    ctx2.clearRect(0, 0, chart.width, chart.height);
    ctx2.strokeStyle = '#90caf9';
    ctx2.lineWidth = 1.8;
    ctx2.beginPath();
    data.forEach((value, index) => {
      const x = index / Math.max(1, data.length - 1) * chart.width;
      const y = chart.height - ((value - min) / range) * (chart.height - 20) - 8;
      index === 0 ? ctx2.moveTo(x, y) : ctx2.lineTo(x, y);
    });
    ctx2.stroke();
  }, 40);
};

// ============================================================
// COMPANY DETAIL
// ============================================================
function formatCompanyChartValue(mode, value){
  if(mode === 'price') return formatMoneyCompact(value);
  if(mode === 'revenue') return formatMoneyCompact(value);
  if(mode === 'demand') return `${Number(value).toFixed(1)}`;
  return `${Math.round(value)}`;
}

function getCompanyChartPayload(company, mode=S.companyChartMode, range=S.companyChartRange){
  const chartMode = ['price','revenue','demand'].includes(mode) ? mode : 'price';
  const sourceData = chartMode === 'price'
    ? company.hist
    : chartMode === 'revenue'
      ? company.revenueHist
      : company.demandHist;
  const fallback = chartMode === 'price' ? company.price : chartMode === 'revenue' ? company.revenue : company.demand;
  const rawData = Array.isArray(sourceData) ? (range >= 9999 ? sourceData.slice() : sourceData.slice(-range)) : [];
  const fallbackValue = typeof fallback === 'number' ? fallback : 0;
  const data = rawData.length ? rawData.slice() : [fallbackValue];
  data[data.length - 1] = fallbackValue;
  if(data.length < 2) data.push(fallbackValue);
  const historyLabels = Array.isArray(S.history?.weeks) ? S.history.weeks : [];
  const rawLabels = historyLabels.length >= data.length ? historyLabels.slice(-data.length) : [];
  const labels = data.map((_, index) => {
    if(rawLabels[index]) return rawLabels[index];
    const relativeIndex = data.length - index - 1;
    if(relativeIndex === 0) return formatWeekText();
    return S.language === 'en' ? `${relativeIndex}w ago` : `${relativeIndex}週前`;
  });
  const color = chartMode === 'price' ? sectors[company.sector]?.color || '#4fc3f7' : chartMode === 'revenue' ? '#4ecca3' : '#ffd180';
  const label = chartMode === 'price' ? '株価' : chartMode === 'revenue' ? '売上' : '需要';
  return {mode:chartMode, data, labels, color, label};
}

function bindCompanyChartHover(payload){
  const wrap = document.getElementById('companyChartWrap');
  const readout = document.getElementById('companyChartHoverReadout');
  const line = document.getElementById('companyChartCursorLine');
  const marker = document.getElementById('companyChartPointMarker');
  const chartCanvas = document.getElementById('companyChart');
  if(!wrap || !readout || !line || !marker || !payload?.data?.length) return;
  const min = Math.min(...payload.data);
  const max = Math.max(...payload.data);
  const range = max - min || 1;
  const setIndex = (idx, visible=true) => {
    const safe = clamp(idx, 0, payload.data.length - 1);
    const xPct = payload.data.length <= 1 ? 0 : (safe / Math.max(1, payload.data.length - 1)) * 100;
    const yPct = 100 - (((payload.data[safe] - min) / range) * 84 + 8);
    readout.innerHTML = `<span>${payload.labels[safe] || '-'}</span><span style="color:${payload.color};font-weight:700">${formatCompanyChartValue(payload.mode, payload.data[safe])}</span>`;
    line.style.left = `${xPct}%`;
    line.style.opacity = visible ? '1' : '0';
    marker.style.left = `${xPct}%`;
    marker.style.top = `${yPct}%`;
    marker.style.opacity = visible ? '1' : '0';
    marker.style.borderColor = payload.color;
  };
  const handleMove = event => {
    const rect = wrap.getBoundingClientRect();
    const pct = clamp((event.clientX - rect.left) / Math.max(1, rect.width), 0, 1);
    const index = Math.round(pct * Math.max(0, payload.data.length - 1));
    setIndex(index, true);
  };
  wrap.onmousemove = handleMove;
  if(chartCanvas) chartCanvas.onmousemove = handleMove;
  wrap.onmouseleave = () => setIndex(payload.data.length - 1, true);
  if(chartCanvas) chartCanvas.onmouseleave = () => setIndex(payload.data.length - 1, true);
  setIndex(payload.data.length - 1, true);
}

function drawCompanyDetailChart(company){
  const chartCanvas = document.getElementById('companyChart');
  if(!chartCanvas) return;
  const ctx = chartCanvas.getContext('2d');
  const payload = getCompanyChartPayload(company, S.companyChartMode, S.companyChartRange);
  const w = chartCanvas.width, h = chartCanvas.height;
  const min = Math.min(...payload.data);
  const max = Math.max(...payload.data);
  const range = max - min || 1;
  ctx.clearRect(0, 0, w, h);
  ctx.strokeStyle = payload.color;
  ctx.lineWidth = 1.5;
  ctx.beginPath();
  payload.data.forEach((value, index) => {
    const x = index / Math.max(1, payload.data.length - 1) * w;
    const y = h - (value - min) / range * (h - 10) - 5;
    index === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y);
  });
  ctx.stroke();
  ctx.lineTo(w, h);
  ctx.lineTo(0, h);
  ctx.closePath();
  ctx.fillStyle = payload.color + '15';
  ctx.fill();
  ctx.fillStyle = 'rgba(255,255,255,.8)';
  ctx.font = '12px sans-serif';
  ctx.fillText(`${payload.label} / ${S.companyChartRange >= 9999 ? '全期間' : `${Math.round(S.companyChartRange/52)}年`}`, 10, 18);
  bindCompanyChartHover(payload);
}

window.openCompanyDetail = function(name){
  const c = companies.find(co => co.name === name);
  if(!c) return;
  S.techFocusCompany = c.name;
  if(c.prefecture){
    S.japanFocusPrefecture = c.prefecture;
    updateMapHud();
  }
  syncCompanyRoadmap(c);
  const sec = sectors[c.sector];
  const modal = document.getElementById('actionModal');
  const modalCard = modal.querySelector('.modal');
  if(modalCard) modalCard.classList.remove('modal-world-country','modal-cyber','modal-finance','modal-prefecture');
  if(modalCard) modalCard.classList.add('modal-company');
  document.getElementById('modalTitle').textContent = `${sec.icon} ${c.name}`;
  
  const owned = S.ownedStocks[c.name] || 0;
  const ownedRatio = clamp(owned / c.sharesOutstanding * 100, 0, 100);
  const costBasis = S.stockCostBasis?.[c.name] || 0;
  const currentValue = owned * c.price;
  const unrealized = currentValue - costBasis;
  const avgCost = owned > 0 ? costBasis / owned : 0;
  const annualPayout = Math.round(c.revenue * c.margin * 0.22 * (owned / c.sharesOutstanding));
  const researchProjectsForCompany = researchCatalog.filter(project => project.company === c.name).map(project => project.name);
  const support = getCompanyAnnualSupport(c.name);
  const activeTrack = getCompanyTrack(c);
  const activeRoadmap = c.roadmap?.[c.roadmapIndex];
  const chartPayload = getCompanyChartPayload(c, S.companyChartMode, S.companyChartRange);
  const allTimeHigh = Math.max(c.base || c.price, ...(Array.isArray(c.hist) ? c.hist : [c.price]));
  const atPeak = c.price >= allTimeHigh * 0.985;
  
  // Period returns
  function getReturn(weeks){
    if(c.hist.length < weeks) return 'N/A';
    const old = c.hist[c.hist.length - weeks];
    return ((c.price - old) / old * 100).toFixed(1) + '%';
  }
  
  const needs = c.needs.map(n => {
    const r = resources[n];
    if(!r) return `<span style="color:#888">${n}</span>`;
    const ratio = r.max ? r.amount / r.max : 0;
    const ok = r.amount > 0;
    const label = ok ? ratio < 0.08 ? '⚠' : '✅' : '❌';
    const jump = tradePartners.some(partner => partner.sells.includes(n)) ? `<button class="resource-jump-btn" onclick="openTradeForResource('${n}')">貿易へ</button>` : '';
    return `<span style="display:inline-flex;align-items:center;gap:4px;color:${ok?'#4ecca3':'#e94560'}">${r.name}${label}${jump}</span>`;
  }).join(', ');

  let html = `${atPeak ? `<div class="company-peak-banner">🔥 最高値圏に到達。市場の期待が最高潮です。</div>` : ''}<div class="detail-grid">
    <div class="dg-item"><div class="dg-label">セクター</div><div class="dg-value" style="color:${sec.color}">${sec.name}</div></div>
    <div class="dg-item"><div class="dg-label">地域</div><div class="dg-value">${regions[c.region]?.name||'不明'}</div></div>
    <div class="dg-item"><div class="dg-label">現在株価</div><div class="dg-value">${formatMoneyCompact(c.price)}</div></div>
    <div class="dg-item"><div class="dg-label">保有株数</div><div class="dg-value">${owned}株 (${formatMoneyCompact(owned*c.price)})</div></div>
    <div class="dg-item"><div class="dg-label">発行株数</div><div class="dg-value">${c.sharesOutstanding.toLocaleString()}株</div></div>
    <div class="dg-item"><div class="dg-label">保有率</div><div class="dg-value">${ownedRatio.toFixed(2)}%</div></div>
    <div class="dg-item"><div class="dg-label">年間売上</div><div class="dg-value">${formatMoneyCompact(c.revenue)}</div></div>
    <div class="dg-item"><div class="dg-label">推定利益率</div><div class="dg-value">${(c.margin*100).toFixed(1)}%</div></div>
    <div class="dg-item"><div class="dg-label">年次配当見込み</div><div class="dg-value">${formatMoneyCompact(annualPayout)}</div></div>
    <div class="dg-item"><div class="dg-label">平均取得単価</div><div class="dg-value">${owned > 0 ? `${formatMoneyCompact(Math.round(avgCost))}` : '-'}</div></div>
    <div class="dg-item"><div class="dg-label">今売ると損益</div><div class="dg-value" style="color:${unrealized >= 0 ? '#4ecca3' : '#e94560'}">${owned > 0 ? `${unrealized >= 0 ? '+' : ''}${formatMoneyCompact(unrealized)}` : '-'}</div></div>
    <div class="dg-item"><div class="dg-label">研究協力</div><div class="dg-value">${ownedRatio >= 100 ? '可能' : '100%保有で解放'}</div></div>
    <div class="dg-item"><div class="dg-label">国への好感度</div><div class="dg-value">${Math.round(c.favorability)} / 100</div></div>
    <div class="dg-item"><div class="dg-label">会社技術深度</div><div class="dg-value">${Math.round(c.techDepth)}</div></div>
    <div class="dg-item"><div class="dg-label">需要指数</div><div class="dg-value">${Number(c.demand || c.baseDemand || 100).toFixed(1)}</div></div>
  </div>
  <p style="font-size:10px;color:#aaa;margin:4px 0">${c.desc}</p>
  ${owned > 0 ? `<p style="font-size:11px;color:${unrealized >= 0 ? '#8ff0bf' : '#ff9ca8'};margin:4px 0">保有評価額 ${formatMoneyCompact(currentValue)} / 取得原価 ${formatMoneyCompact(costBasis)} / 現在損益 ${unrealized >= 0 ? '+' : ''}${formatMoneyCompact(unrealized)}</p>` : ''}
  <p style="font-size:10px;color:#ffd180;margin:2px 0">${support > 0 ? `年間支援: ${formatMoneyCompact(support)}` : '年間支援なし'} / 現在ルート: ${activeTrack ? `${activeTrack.name}` : '標準開発'} / 次の段階: ${activeRoadmap ? `${activeRoadmap.name} (${Math.round(c.techProgress)} / ${activeRoadmap.threshold})` : '全達成'}</p>
  ${researchProjectsForCompany.length ? `<p style="font-size:10px;color:#90caf9;margin:2px 0">関連研究: ${researchProjectsForCompany.join(' / ')}</p>` : ''}
  <p style="font-size:10px;margin:2px 0">必要資源: ${needs || 'なし'}</p>
  <div style="margin:8px 0">
    <div style="font-size:10px;font-weight:bold;margin-bottom:4px">📊 期間別リターン</div>
    <div style="display:flex;gap:8px;flex-wrap:wrap;font-size:9px">
      <span>3ヶ月: ${getReturn(13)}</span>
      <span>5ヶ月: ${getReturn(22)}</span>
      <span>2年: ${getReturn(104)}</span>
      <span>5年: ${getReturn(260)}</span>
      <span>10年: ${getReturn(520)}</span>
    </div>
  </div>
  <div class="range-group">
    ${[
      [52,'1年'],[156,'3年'],[260,'5年'],[520,'10年'],[9999,'全期間']
    ].map(([range,label]) => `<button class="btn btn-sm ${S.companyChartRange===range?'active':''}" onclick="setCompanyChartState('${c.name}', ${range}, '${S.companyChartMode}')">${label}</button>`).join('')}
  </div>
  <div class="range-group">
    <button class="btn btn-sm ${S.companyChartMode==='price'?'active':''}" onclick="setCompanyChartState('${c.name}', ${S.companyChartRange}, 'price')">株価</button>
    <button class="btn btn-sm ${S.companyChartMode==='revenue'?'active':''}" onclick="setCompanyChartState('${c.name}', ${S.companyChartRange}, 'revenue')">売上</button>
    <button class="btn btn-sm ${S.companyChartMode==='demand'?'active':''}" onclick="setCompanyChartState('${c.name}', ${S.companyChartRange}, 'demand')">需要</button>
  </div>
  <div class="history-hover-readout" id="companyChartHoverReadout"><span>${chartPayload.labels[chartPayload.labels.length - 1] || formatWeekText()}</span><span style="color:${chartPayload.color};font-weight:700">${formatCompanyChartValue(chartPayload.mode, chartPayload.data[chartPayload.data.length - 1] || 0)}</span></div>
  <div class="history-chart-wrap" id="companyChartWrap" style="height:250px">
    <div class="history-cursor-line" id="companyChartCursorLine"></div>
    <div class="history-point-marker" id="companyChartPointMarker"></div>
    <canvas id="companyChart" width="860" height="250" style="position:absolute;inset:0;width:100%;height:100%;background:rgba(0,0,0,.2);border-radius:6px"></canvas>
  </div>
  <div style="display:flex;gap:4px;flex-wrap:wrap;margin-top:8px">
    <button class="btn btn-green" onclick="buyStock('${c.name}',1)">1株購入 (${formatMoneyCompact(c.price)})</button>
    <button class="btn btn-green" onclick="buyStock('${c.name}',10)">10株購入</button>
    <button class="btn btn-green" onclick="buyStock('${c.name}',50)">50株購入</button>
    <button class="btn btn-green" onclick="buyStock('${c.name}',100)">100株購入</button>
    <button class="btn btn-green" onclick="buyStock('${c.name}',250)">250株購入</button>
    <button class="btn btn-green" onclick="buyStock('${c.name}',500)">500株購入</button>
    <button class="btn btn-green" onclick="buyStock('${c.name}',1000)">1000株購入</button>
    <button class="btn btn-green" onclick="buyStockAll('${c.name}')">全部買う</button>
    <button class="btn" onclick="sellStock('${c.name}',1)" ${owned<1?'disabled':''}>1株売却</button>
    <button class="btn" onclick="sellStock('${c.name}',10)" ${owned<10?'disabled':''}>10株売却</button>
    <button class="btn" onclick="sellStock('${c.name}',100)" ${owned<100?'disabled':''}>100株売却</button>
    <button class="btn" onclick="sellStock('${c.name}',owned)" ${owned<1?'disabled':''}>全売却(${owned})</button>
  </div>
  <div class="section" style="margin-top:8px"><h3>🤝 企業工作</h3>
    <div class="compact-note">年間予算とは別枠で非公式資金を渡して好感度を上げられます。露見すると支持率と安定度に響きます。</div>
    <div class="save-actions" style="margin-top:8px">
      <button class="btn btn-sm btn-yellow" onclick="bribeCompany('${c.name}', 3000)">賄賂 ${formatMoneyCompact(3000)}</button>
      <button class="btn btn-sm btn-yellow" onclick="bribeCompany('${c.name}', 7000)">賄賂 ${formatMoneyCompact(7000)}</button>
      <button class="btn btn-sm btn-yellow" onclick="bribeCompany('${c.name}', 12000)">賄賂 ${formatMoneyCompact(12000)}</button>
    </div>
  </div>
  <div class="section" style="margin-top:8px"><h3>🧠 技術方針</h3>
    <div class="compact-note">${activeTrack ? activeTrack.desc : '標準開発では、その会社本来の技術ロードマップを順番に進めます。'}。切り替えても各方針ごとの進捗は保存され、選んでいる方針だけが進行します。</div>
    <div class="save-actions" style="margin-top:8px">
      <button class="btn btn-sm ${!c.selectedTechTrack?'active':''}" onclick="selectCompanyTechTrack('${c.name}','')">標準</button>
      ${(c.techTracks || []).map(track => `<button class="btn btn-sm ${c.selectedTechTrack===track.id?'active':''}" onclick="selectCompanyTechTrack('${c.name}','${track.id}')">${track.name}</button>`).join('')}
    </div>
  </div>
  <div class="section" style="margin-top:8px"><h3>🧭 会社ロードマップ</h3>
    ${(c.roadmap || []).map((stage, index) => `<div class="roadmap-stage ${index < c.roadmapIndex ? 'done' : index === c.roadmapIndex ? 'active' : ''}">
      <strong>${stage.name}</strong><br>${stage.note}${stage.announce ? '<br>重要節目' : ''}<br>${index < c.roadmapIndex ? '達成済' : index === c.roadmapIndex ? `進行中 ${Math.round(c.techProgress)} / ${stage.threshold}` : `必要進捗 ${stage.threshold}`}
    </div>`).join('')}
  </div>`;
  
  document.getElementById('modalContent').innerHTML = html;
  modal.classList.add('show');
  
  // Draw chart
  setTimeout(()=>{
    drawCompanyDetailChart(c);
  }, 50);
};

window.setCompanyChartState = function(name, range, mode){
  S.companyChartRange = range;
  S.companyChartMode = mode;
  openCompanyDetail(name);
};

window.buyStockAll = function(name){
  const company = getCompanyByName(name);
  if(!company) return;
  const owned = S.ownedStocks[name] || 0;
  const maxQty = Math.max(0, Math.min(company.sharesOutstanding - owned, Math.floor(S.money / Math.max(1, company.price))));
  if(maxQty <= 0){ addEvent('❌ 一括購入できる資金か残株がありません', 'bad'); return; }
  buyStock(name, maxQty);
};

window.buyStock = function(name, qty){
  const c = companies.find(co => co.name === name);
  const owned = S.ownedStocks[name] || 0;
  qty = Math.min(qty, Math.max(0, c.sharesOutstanding - owned));
  if(qty <= 0){ addEvent('ℹ これ以上は取得できません',''); return; }
  const cost = c.price * qty;
  if(S.money < cost){ addEvent('❌ 資金不足','bad'); return; }
  S.money -= cost;
  S.ownedStocks[name] = owned + qty;
  S.stockCostBasis[name] = (S.stockCostBasis[name] || 0) + cost;
  c.favorability = Math.min(100, (c.favorability || 50) + Math.max(0.4, qty / Math.max(40, c.sharesOutstanding / 180)));
  addEvent(`📈 ${name} ${qty}株購入 (${formatMoneyCompact(cost)})`,'good');
  updateUI();
  openCompanyDetail(name);
};

window.bribeCompany = function(name, amount){
  const company = getCompanyByName(name);
  if(!company) return;
  if(S.money < amount){ addEvent('❌ 資金不足', 'bad'); return; }
  S.money -= amount;
  const gain = amount <= 3000 ? rand(4, 8) : amount <= 7000 ? rand(8, 15) : rand(14, 24);
  company.favorability = clamp((company.favorability || 50) + gain, 0, 100);
  company.revenue *= 1 + gain * 0.0009;
  company.price = Math.round(company.price * (1 + gain * 0.0022));
  const scandalChance = clamp(0.08 + amount / 26000 - Math.max(0, company.favorability - 60) * 0.001, 0.06, 0.44);
  if(Math.random() < scandalChance){
    const approvalHit = Math.round(rand(2, 7));
    S.approval = Math.max(0, S.approval - approvalHit);
    S.stability = Math.max(0, S.stability - approvalHit * 0.7);
    S.briberyExposure = clamp((S.briberyExposure || 0) + approvalHit * 2.4, 0, 100);
    addEvent(`🕵 ${company.name} への賄賂が露見。支持率 -${approvalHit}`, 'bad');
  } else {
    addEvent(`🤝 ${company.name} へ非公式資金を投入。好感度 +${Math.round(gain)}`, '');
  }
  updateUI();
  openCompanyDetail(name);
};

window.sellStock = function(name, qty){
  const c = companies.find(co => co.name === name);
  const owned = S.ownedStocks[name] || 0;
  if(owned < qty) qty = owned;
  if(qty <= 0) return;
  const proceeds = c.price * qty;
  const basis = S.stockCostBasis[name] || 0;
  const avgCost = owned > 0 ? basis / owned : 0;
  const realized = proceeds - avgCost * qty;
  S.money += proceeds;
  S.ownedStocks[name] = owned - qty;
  if(S.ownedStocks[name] > 0) S.stockCostBasis[name] = Math.max(0, basis - avgCost * qty);
  else delete S.stockCostBasis[name];
  if(S.ownedStocks[name] <= 0) delete S.ownedStocks[name];
  c.favorability = Math.max(0, (c.favorability || 50) - Math.max(0.25, qty / Math.max(70, c.sharesOutstanding / 220)));
  addEvent(`📉 ${name} ${qty}株売却 (+${formatMoneyCompact(proceeds)} / 実現損益 ${realized >= 0 ? '+' : ''}${formatMoneyCompact(Math.round(realized))})`,'good');
  updateUI();
  openCompanyDetail(name);
};

// ============================================================
// PRICE BAR (selectable)
// ============================================================
function updatePriceBar(){
  const bar = document.getElementById('priceBar');
  bar.innerHTML = '';
  getBudgetSupportEntries().forEach(entry => {
    const company = getCompanyByName(entry.company);
    if(!company) return;
    const prev = company.prevPrice || company.price;
    const ch = ((company.price - prev) / Math.max(1, prev) * 100).toFixed(2);
    const up = company.price >= prev;
    const el = document.createElement('div');
    el.className = 'price-item';
    el.innerHTML = `<span class="dot" style="background:${sectors[company.sector]?.color || '#4fc3f7'}"></span>
      <span style="color:#ffd180">予算 ${company.name}</span>
      <span style="color:#fff;font-weight:bold">¥${Math.round(company.price).toLocaleString()}</span>
      <span class="${up?'price-up':'price-down'}">${up?'▲':'▼'}${ch}%</span>`;
    el.onclick = () => {
      S.currentPanel = 'market';
      renderPanel();
      openCompanyDetail(company.name);
    };
    bar.appendChild(el);
  });
  S.trackedPrices.forEach(rk => {
    const r = resources[rk]; if(!r || !r.price) return;
    const prev = r.priceHist.length > 1 ? r.priceHist[r.priceHist.length-2] : r.price;
    const ch = ((r.price - prev) / prev * 100).toFixed(2);
    const up = r.price >= prev;
    const el = document.createElement('div');
    el.className = 'price-item';
    el.innerHTML = `<span class="dot" style="background:${r.color}"></span>
      <span style="color:#ccc">${r.name}</span>
      <span style="color:#fff;font-weight:bold">${formatPrice(r.price)}/${r.unit||'t'}</span>
      <span class="${up?'price-up':'price-down'}">${up?'▲':'▼'}${ch}%</span>
      <span style="color:#555;font-size:8px;margin-left:2px">✕</span>`;
    el.onclick = (e) => {
      if(e.target.textContent === '✕'){
        S.trackedPrices = S.trackedPrices.filter(x => x !== rk);
        updatePriceBar();
      }
    };
    bar.appendChild(el);
  });
  const addBtn = document.createElement('div');
  addBtn.className = 'price-item';
  addBtn.innerHTML = '<span style="color:#555">+ 追加</span>';
  addBtn.onclick = () => openPriceSelectModal();
  bar.appendChild(addBtn);
}

function formatPrice(p){
  if(p >= 10000) return (p/1000).toFixed(1) + 'K';
  if(p >= 100) return p.toFixed(0);
  if(p >= 1) return p.toFixed(2);
  return p.toFixed(3);
}

function openPriceSelectModal(){
  const modal = document.getElementById('actionModal');
  document.getElementById('modalTitle').textContent = '📊 追跡する資源価格を選択';
  let html = '<div style="display:flex;flex-wrap:wrap;gap:4px">';
  Object.entries(resources).forEach(([k,r])=>{
    if(!r.price) return;
    const tracked = S.trackedPrices.includes(k);
    html += `<button class="btn ${tracked?'active':''}" style="font-size:9px;border-color:${r.color}" 
      onclick="toggleTracked('${k}')">${r.name} ${tracked?'✓':''}</button>`;
  });
  // Unique materials
  S.discoveredMaterials.forEach(dm=>{
    const mat = [...uniqueMaterials, ...spaceMaterials].find(m=>m.id===dm.id);
    if(mat && resources[dm.id]){
      const tracked = S.trackedPrices.includes(dm.id);
      html += `<button class="btn ${tracked?'active':''}" style="font-size:9px;border-color:${mat.color}" 
        onclick="toggleTracked('${dm.id}')">✦${mat.name} ${tracked?'✓':''}</button>`;
    }
  });
  html += '</div>';
  document.getElementById('modalContent').innerHTML = html;
  modal.classList.add('show');
}

window.toggleTracked = function(k){
  if(S.trackedPrices.includes(k)) S.trackedPrices = S.trackedPrices.filter(x=>x!==k);
  else S.trackedPrices.push(k);
  updatePriceBar();
  openPriceSelectModal();
};

// ============================================================
// RIGHT PANEL RENDERING
// ============================================================
document.getElementById('panelTabs').addEventListener('click', e=>{
  if(!e.target.dataset.panel) return;
  switchPanelView(e.target.dataset.panel);
});

document.getElementById('topBar').addEventListener('click', e => {
  const stat = e.target.closest('.stat[data-stat]');
  if(!stat) return;
  handleTopStatCardClick(stat.dataset.stat);
});

function getTopStatHistoryInfo(stat){
  const map = {
    money:{metric:'money', color:'#4fc3f7', label:'国庫'},
    polCap:{metric:'polCap', color:'#ce93d8', label:'政治資本'},
    pop:{metric:'pop', color:'#81c784', label:'人口'},
    gdp:{metric:'gdp', color:'#4ecca3', label:'GDP'},
    approval:{metric:'approval', color:'#ffd54f', label:'支持率'},
    bubble:{metric:'bubble', color:'#e94560', label:'バブル'},
    yen:{metric:'yen', color:'#80cbc4', label:'円指数'},
    tech:{metric:'tech', color:'#90caf9', label:'技術力'},
    defense:{metric:'def', color:'#64b5f6', label:'防衛力'},
    cyber:{metric:'cyber', color:'#7e57c2', label:'サイバー'},
    nuclear:{metric:'nuclear', color:'#ef9a9a', label:'核戦力'},
  };
  return map[stat] || null;
}

function renderTopStatQuickChart(stat){
  const info = getTopStatHistoryInfo(stat);
  if(!info) return '';
  const series = S.history?.[info.metric] || [];
  const labels = S.history?.weeks || [];
  if(series.length < 2) return '';
  const usedSeries = series.slice(-156);
  const usedLabels = labels.slice(-156);
  const svg = buildMiniChartSvg(usedSeries, info.color, {full:true, lineWidth:1}).replace('class="mini-chart"', 'class="history-chart-svg"');
  return `<div class="section">
    <div class="history-hover-readout" id="topStatHoverReadout"><span>${usedLabels[usedLabels.length - 1] || '-'}</span><span style="color:${info.color};font-weight:700">${formatHistoryValue(info.metric, usedSeries[usedSeries.length - 1])}</span></div>
    <div class="history-chart-wrap top-stat-chart-wrap" id="topStatChartWrap">${svg}<div class="history-cursor-line" id="topStatCursorLine"></div><div class="history-point-marker" id="topStatPointMarker"></div></div>
    <div class="save-actions" style="margin-top:8px"><button class="btn btn-sm btn-blue" onclick="openHistoryExplorer('${info.metric}')">${info.label} の履歴を開く</button></div>
  </div>`;
}

function bindTopStatQuickChart(stat){
  const info = getTopStatHistoryInfo(stat);
  if(!info) return;
  const wrap = document.getElementById('topStatChartWrap');
  const readout = document.getElementById('topStatHoverReadout');
  const line = document.getElementById('topStatCursorLine');
  const marker = document.getElementById('topStatPointMarker');
  const series = (S.history?.[info.metric] || []).slice(-156);
  const labels = (S.history?.weeks || []).slice(-156);
  if(!wrap || !readout || !line || !marker || !series.length) return;
  const meta = getHistoryMeta(info.metric);
  const bounds = getChartSeriesBounds(series, meta);
  const min = bounds.min;
  const max = bounds.max;
  const range = max - min || 1;
  const setIndex = (index, visible=true) => {
    const idx = clamp(index, 0, series.length - 1);
    const pos = getSvgChartMarkerPosition(wrap, idx, series.length, series[idx], min, range);
    readout.innerHTML = `<span>${labels[idx] || '-'}</span><span style="color:${info.color};font-weight:700">${formatHistoryValue(info.metric, series[idx])}</span>`;
    line.style.left = `${pos.x}px`;
    line.style.opacity = visible ? '1' : '0';
    marker.style.left = `${pos.x}px`;
    marker.style.top = `${pos.y}px`;
    marker.style.borderColor = info.color;
    marker.style.opacity = visible ? '1' : '0';
  };
  const moveHandler = event => {
    const touch = event.touches?.[0] || event.changedTouches?.[0];
    const clientX = touch ? touch.clientX : event.clientX;
    const rect = wrap.getBoundingClientRect();
    const pct = clamp((clientX - rect.left) / Math.max(1, rect.width), 0, 1);
    const index = Math.round(pct * Math.max(0, series.length - 1));
    setIndex(index, true);
  };
  setIndex(series.length - 1, true);
  wrap.addEventListener('mousemove', moveHandler);
  wrap.addEventListener('mouseenter', moveHandler);
  wrap.addEventListener('touchstart', moveHandler, {passive:true});
  wrap.addEventListener('touchmove', moveHandler, {passive:true});
  wrap.addEventListener('mouseleave', () => setIndex(series.length - 1, true));
}

function getPopulationDonutItems(){
  ensureAllocationState();
  const palette = ['#4ecca3','#4fc3f7','#ffd54f','#ce93d8','#ff8a65','#80cbc4','#90caf9','#f06292','#aed581','#ffb74d','#9575cd'];
  const statEntries = Object.entries(S.prefectureStats || {})
    .map(([prefectureId, stat]) => ({
      prefectureId,
      name:prefectureMap[prefectureId]?.name || prefectureId,
      value:Math.max(0, Number(stat?.population) || 0),
    }))
    .filter(item => item.value > 0)
    .sort((a, b) => b.value - a.value);
  const items = statEntries.slice(0, 10).map((item, index) => ({...item, color:palette[index % palette.length]}));
  const remaining = statEntries.slice(10).reduce((sum, item) => sum + item.value, 0);
  if(remaining > 0){
    items.push({
      prefectureId:'others',
      name:S.language === 'en' ? 'Others' : 'その他',
      value:remaining,
      color:'rgba(255,255,255,.26)',
    });
  }
  return items;
}

function renderPopulationBreakdownPanel(){
  const items = getPopulationDonutItems();
  if(!items.length){
    return `<div class="section"><div class="compact-note">${S.language === 'en' ? 'No prefecture population data.' : '県人口データがありません。'}</div></div>`;
  }
  const total = items.reduce((sum, item) => sum + item.value, 0);
  const demoBuilder = window.getNationalDemographicSummary;
  const pyramidBuilder = window.buildPopulationPyramidSvg;
  const demo = typeof demoBuilder === 'function'
    ? demoBuilder()
    : {children:0, workingAge:0, seniors:0, population:total, birthRate:Number(S.birthRate || 0), childShare:0, workingShare:0, seniorShare:0, oldAgeDependency:0};
  const legend = items.map(item => `<div class="finance-legend-row"><span class="finance-legend-swatch" style="background:${item.color}"></span><span>${item.name}</span><span>${((item.value / Math.max(1, total)) * 100).toFixed(1)}%</span><strong>${formatPopulationCompact(item.value, 1)}</strong></div>`).join('');
  const demoCards = `
    <div class="detail-grid" style="margin-top:8px">
      <div class="dg-item"><div class="dg-label">出生率</div><div class="dg-value">${demo.birthRate.toFixed(2)}</div></div>
      <div class="dg-item"><div class="dg-label">子ども</div><div class="dg-value">${formatPopulationCompact(demo.children, 1)} (${(demo.childShare * 100).toFixed(1)}%)</div></div>
      <div class="dg-item"><div class="dg-label">生産年齢</div><div class="dg-value">${formatPopulationCompact(demo.workingAge, 1)} (${(demo.workingShare * 100).toFixed(1)}%)</div></div>
      <div class="dg-item"><div class="dg-label">高齢者</div><div class="dg-value">${formatPopulationCompact(demo.seniors, 1)} (${(demo.seniorShare * 100).toFixed(1)}%)</div></div>
      <div class="dg-item"><div class="dg-label">高齢化率</div><div class="dg-value">${(demo.seniorShare * 100).toFixed(1)}%</div></div>
      <div class="dg-item"><div class="dg-label">老年扶養比率</div><div class="dg-value">${(demo.oldAgeDependency * 100).toFixed(1)}%</div></div>
    </div>`;
  return `<div class="section">
    <h3>${S.language === 'en' ? 'Population Share By Prefecture' : '都道府県別人口構成'}</h3>
    <div class="finance-grid">
      <div class="finance-card">
        ${buildDonutChartSvg(items, {
          centerTop:S.language === 'en' ? 'Population' : '人口',
          centerBottom:formatPopulationCompact(total, 1)
        })}
      </div>
      <div class="finance-card">
        <div class="finance-legend">${legend}</div>
      </div>
    </div>
  </div><div class="section">
    <h3>${S.language === 'en' ? 'Population Pyramid' : '人口ピラミッド'}</h3>
    <div class="finance-grid">
      <div class="finance-card">${typeof pyramidBuilder === 'function' ? pyramidBuilder(demo) : `<div class="compact-note">${S.language === 'en' ? 'Demographic chart is preparing.' : '人口ピラミッドを準備中です。'}</div>`}</div>
      <div class="finance-card">${demoCards}</div>
    </div>
  </div>`;
}

function renderTopStatActionButtons(stat){
  const actions = {
    money:[
      {label:'年間予算', onclick:`openAction('budget')`},
      {label:'日銀', onclick:`closeModal(); switchPanelView('bank')`},
      {label:'履歴', onclick:`openHistoryExplorer('money')`},
    ],
    polCap:[
      {label:'法律', onclick:`closeModal(); switchPanelView('law')`},
      {label:'年間予算', onclick:`openAction('budget')`},
      {label:'国家目標', onclick:`closeModal(); switchPanelView('goals')`},
    ],
    pop:[
      {label:'資源配分', onclick:`closeModal(); switchPanelView('resources')`},
      {label:'開発', onclick:`closeModal(); switchPanelView('develop')`},
      {label:'履歴', onclick:`openHistoryExplorer('pop')`},
    ],
    gdp:[
      {label:'市場', onclick:`closeModal(); switchPanelView('market')`},
      {label:'国家目標', onclick:`closeModal(); switchPanelView('goals')`},
      {label:'履歴', onclick:`openHistoryExplorer('gdp')`},
    ],
    approval:[
      {label:'政策', onclick:`openAction('budget')`},
      {label:'ニュース', onclick:`closeModal(); switchPanelView('news')`},
      {label:'履歴', onclick:`openHistoryExplorer('approval')`},
    ],
    bubble:[
      {label:'日銀', onclick:`closeModal(); switchPanelView('bank')`},
      {label:'履歴', onclick:`openHistoryExplorer('bubble')`},
    ],
    yen:[
      {label:'貿易', onclick:`closeModal(); switchPanelView('trade')`},
      {label:'比較', onclick:`closeModal(); switchPanelView('compare')`},
      {label:'履歴', onclick:`openHistoryExplorer('yen')`},
    ],
    tech:[
      {label:'研究', onclick:`closeModal(); switchPanelView('research')`},
      {label:'宇宙', onclick:`closeModal(); switchPanelView('space')`},
      {label:'サイバー', onclick:`closeModal(); switchPanelView('cyber')`},
    ],
    defense:[
      {label:'戦力', onclick:`closeModal(); switchPanelView('nuke')`},
      {label:'世界', onclick:`closeModal(); switchPanelView('world')`},
      {label:'比較', onclick:`closeModal(); switchPanelView('compare')`},
    ],
    cyber:[
      {label:'サイバー', onclick:`closeModal(); switchPanelView('cyber')`},
      {label:'研究', onclick:`closeModal(); switchPanelView('research')`},
      {label:'世界', onclick:`closeModal(); switchPanelView('world')`},
    ],
    nuclear:[
      {label:'戦力', onclick:`closeModal(); switchPanelView('nuke')`},
      {label:'世界', onclick:`closeModal(); switchPanelView('world')`},
    ],
  }[stat] || [];
  return actions.length
    ? `<div class="save-actions" style="margin-top:10px">${actions.map(action => `<button class="btn btn-sm btn-blue" onclick="${action.onclick}">${action.label}</button>`).join('')}</div>`
    : '';
}

function handleTopStatCardClick(stat){
  try{
    openTopStatDetail(stat);
  } catch(error){
    reportRuntimeError('openTopStatDetail', error);
  }
}
window.handleTopStatCardClick = handleTopStatCardClick;

function openTopStatDetail(stat){
  const modal = document.getElementById('actionModal');
  const title = document.getElementById('modalTitle');
  const content = document.getElementById('modalContent');
  modal.dataset.action = 'top-stat';
  modal.dataset.topStat = stat;
  const modalCard = modal.querySelector('.modal');
  if(modalCard) modalCard.classList.remove('modal-finance');
  const b = S.weeklyBreakdown;
  const detailMap = {
    money:{
      title:'💰 国庫の内訳',
      body:() => `${renderFinanceBreakdownPanel()}<div class="detail-grid" style="margin-top:14px">
        <div class="dg-item"><div class="dg-label">税収</div><div class="dg-value">${formatMoneyCompact(b.tax)}</div></div>
        <div class="dg-item"><div class="dg-label">予算支出</div><div class="dg-value">${formatMoneyCompact(-b.budget)}</div></div>
        <div class="dg-item"><div class="dg-label">貿易収支</div><div class="dg-value">${formatMoneyCompact(b.trade)}</div></div>
        <div class="dg-item"><div class="dg-label">配当</div><div class="dg-value">${formatMoneyCompact(b.dividend)}</div></div>
        <div class="dg-item"><div class="dg-label">宇宙収入</div><div class="dg-value">${formatMoneyCompact(b.space)}</div></div>
        <div class="dg-item"><div class="dg-label">利払い</div><div class="dg-value">${formatMoneyCompact(-b.interest)}</div></div>
        <div class="dg-item"><div class="dg-label">元本返済</div><div class="dg-value">${formatMoneyCompact(-(b.principal || 0))}</div></div>
        <div class="dg-item"><div class="dg-label">企業支援</div><div class="dg-value">${formatMoneyCompact(-b.companySupport)}</div></div>
        <div class="dg-item"><div class="dg-label">公共収入</div><div class="dg-value">${formatMoneyCompact(b.publicRevenue || 0)}</div></div>
        <div class="dg-item"><div class="dg-label">公共運転費</div><div class="dg-value">${formatMoneyCompact(-(b.operations || 0))}</div></div>
        <div class="dg-item"><div class="dg-label">社会保障</div><div class="dg-value">${formatMoneyCompact(-(b.socialSecurity || 0))}</div></div>
        <div class="dg-item"><div class="dg-label">医療費</div><div class="dg-value">${formatMoneyCompact(-(b.healthcare || 0))}</div></div>
        <div class="dg-item"><div class="dg-label">子育て</div><div class="dg-value">${formatMoneyCompact(-(b.childcare || 0))}</div></div>
        <div class="dg-item"><div class="dg-label">行政費</div><div class="dg-value">${formatMoneyCompact(-(b.administration || 0))}</div></div>
        <div class="dg-item"><div class="dg-label">調整</div><div class="dg-value">${formatMoneyCompact(b.adjustments || 0)}</div></div>
      </div><div class="compact-note" style="margin-top:8px">現在税率: ${S.taxRate}% / 年次配当は毎年末に一括計上</div>`
    },
    polCap:{title:'🏛 政治資本',body:() => `<div class="compact-note">支持率、安定度、イベント、政策実行で増減します。現在 ${S.polCap.toFixed(1)} / ${S.polCapMax}</div>`},
    pop:{title:'👥 人口',body:() => `${renderPopulationBreakdownPanel()}<div class="compact-note">教育・雇用・生活コスト・災害・交通網・県条例の影響を受けます。現在 ${S.pop.toFixed(1)}万 / 出生率 ${Number(S.birthRate || 0).toFixed(2)}</div>`},
    gdp:{title:'📈 GDP',body:() => `<div class="compact-note">景気波、企業売上、貿易、公共投資、技術力、重大イベントで変化します。現在 ${S.gdp.toFixed(1)}兆</div>`},
    approval:{title:'😊 支持率',body:() => `<div class="compact-note">教育、公共事業、重大イベント、物価、失業率、国庫収支で変化します。現在 ${S.approval.toFixed(1)}%</div>`},
    bubble:{title:'🎈 バブル',body:() => `<div class="compact-note">補助金、投機、海外資金流入、景気過熱で上昇します。発生中は約3年かけて減衰しつつ、崩壊時は強く落ちます。</div>`},
    yen:{title:'💱 円指数',body:() => `<div class="compact-note">貿易収支、技術力、国際関係、負債、重大イベントで動きます。円安は輸出株に追い風、輸入依存業種に逆風です。<br>${tradePartners.slice(0,4).map(partner => `${partner.name} ${Number(partner.currencyRate || 0).toFixed(2)}`).join(' / ')}</div>`},
    tech:{title:'🧠 技術力',body:() => `<div class="compact-note">教育、技術投資、研究、宇宙開発で上がります。研究条件や宇宙・核成功率にも直結します。</div>`},
    defense:{title:'🛡 防衛力',body:() => `<div class="compact-note">軍事予算、核、宇宙、防衛研究で上昇します。外交・安定度・抑止力にも影響します。</div>`},
    cyber:{title:'💻 サイバー指数',body:() => `<div class="compact-note">技術投資や研究で上がります。貿易コスト、相手国AIの攻撃耐性、こちらの攻撃成功率に反映されます。現在 ${Math.round(S.cyberPower)}</div>`},
    nuclear:{title:'☢ 核戦力',body:() => `<div class="compact-note">長期計画でのみ増加します。成功すると防衛力は上がりますが国際関係と支持率が悪化します。</div>`},
  };
  const entry = detailMap[stat];
  if(stat === 'money' && modalCard) modalCard.classList.add('modal-finance');
  title.textContent = entry?.title || '詳細';
  let bodyHtml = '<div class="compact-note">詳細はありません。</div>';
  try{
    if(entry?.body) bodyHtml = entry.body();
  } catch(error){
    reportRuntimeError(`topStat:${stat}`, error);
    bodyHtml = `<div class="compact-note">詳細の表示中に問題が発生しましたが、カードは開きました。</div>`;
  }
  content.innerHTML = `${renderTopStatQuickChart(stat)}${bodyHtml}${renderTopStatActionButtons(stat)}`;
  bindTopStatQuickChart(stat);
  modal.classList.add('show');
}

function getPortfolioValue(){
  return Object.entries(S.ownedStocks).reduce((sum, [name, qty]) => {
    const company = companies.find(c => c.name === name);
    return sum + (company ? company.price * qty : 0);
  }, 0);
}

function getTotalDebt(){
  return S.loans.reduce((sum, loan) => sum + loan.amount, 0);
}

function getAverageRelation(){
  return tradePartners.reduce((sum, partner) => sum + partner.relation, 0) / tradePartners.length;
}

function getAverageCyberTarget(){
  return tradePartners.reduce((sum, partner) => sum + partner.cyberPower, 0) / tradePartners.length;
}

function getShortageCount(){
  return Object.values(resources).filter(resource => resource.max && resource.amount < resource.max * 0.05).length;
}

function getSpaceTechBonus(){
  return S.discoveredMaterials.reduce((bonus, materialRef) => {
    const material = [...uniqueMaterials, ...spaceMaterials].find(m => m.id === materialRef.id);
    return bonus + (material?.effects?.tech ? 0.018 : 0);
  }, 0);
}

function pushHistoryPoint(force=false){
  ensureStateShape();
  const label = `${S.year}W${S.week}`;
  const snapshot = {
    weeks:label,
    money:S.money,
    gdp:S.gdp,
    approval:S.approval,
    bubble:S.bubbleIndex,
    pop:S.pop,
    debt:getTotalDebt(),
    inflation:S.inflation,
    unemployment:S.unemployment,
    stability:S.stability,
    def:S.defPower,
    cyber:S.cyberPower,
    tech:S.techLevel,
    yen:S.yenValue,
    cycle:S.businessCycle,
    unrest:S.socialUnrest,
    ideology:S.ideologyScore,
    polCap:S.polCap,
    nuclear:S.nukePower,
  };
  const latestLabel = S.history.weeks[S.history.weeks.length - 1];
  if(latestLabel === label){
    if(!force) return;
    Object.entries(snapshot).forEach(([key, value]) => {
      if(Array.isArray(S.history[key]) && S.history[key].length){
        S.history[key][S.history[key].length - 1] = value;
      }
    });
    return;
  }
  Object.entries(snapshot).forEach(([key, value]) => {
    S.history[key].push(value);
    if(S.history[key].length > 1040) S.history[key].shift();
  });
}

function buildMiniChartSvg(data, color, options={}){
  if(!data || data.length < 2) return '<div class="mini-chart"></div>';
  const maxPoints = options.full ? data.length : (options.maxPoints || 52);
  const slice = data.slice(-maxPoints);
  const bounds = getChartSeriesBounds(slice, options);
  const min = bounds.min;
  const max = bounds.max;
  const range = max - min || 1;
  const gradientId = `grad-${color.replace('#','')}`;
  const strokeWidth = options.lineWidth || 1.25;
  const points = slice.map((value, index) => {
    const x = slice.length === 1 ? 0 : index / (slice.length - 1) * 100;
    const y = 100 - ((value - min) / range * 84 + 8);
    return `${x.toFixed(2)},${y.toFixed(2)}`;
  }).join(' ');
  const fillPoints = `0,100 ${points} 100,100`;
  const midLine = typeof options.midLineValue === 'number'
    ? (() => {
        const y = 100 - ((options.midLineValue - min) / range * 84 + 8);
        return `<line x1="0" y1="${y.toFixed(2)}" x2="100" y2="${y.toFixed(2)}" stroke="rgba(255,255,255,.16)" stroke-width=".7" stroke-dasharray="3 3"></line>`;
      })()
    : '';
  return `<svg class="mini-chart" viewBox="0 0 100 100" preserveAspectRatio="none">
    <defs><linearGradient id="${gradientId}" x1="0" x2="0" y1="0" y2="1">
      <stop offset="0%" stop-color="${color}" stop-opacity=".28"></stop>
      <stop offset="100%" stop-color="${color}" stop-opacity="0"></stop>
    </linearGradient></defs>
    ${midLine}
    <polyline points="${fillPoints}" fill="url(#${gradientId})" stroke="none"></polyline>
    <polyline points="${points}" fill="none" stroke="${color}" stroke-width="${strokeWidth}" stroke-linecap="round" stroke-linejoin="round"></polyline>
  </svg>`;
}

function getChartSeriesBounds(series, options={}){
  let min = typeof options.fixedMin === 'number' ? options.fixedMin : Math.min(...series);
  let max = typeof options.fixedMax === 'number' ? options.fixedMax : Math.max(...series);
  if(typeof options.fixedMin === 'number' && typeof options.fixedMax === 'number'){
    if(min === max){
      min -= 1;
      max += 1;
    }
    return {min, max};
  }
  const baseRange = max - min;
  const magnitude = Math.max(Math.abs(min), Math.abs(max), 1);
  const padRatio = typeof options.padRatio === 'number' ? options.padRatio : 0.18;
  const padMagnitude = typeof options.padMagnitude === 'number' ? options.padMagnitude : 0.06;
  const minPadding = typeof options.minPadding === 'number' ? options.minPadding : 0;
  let padding = Math.max(minPadding, baseRange * padRatio, magnitude * padMagnitude);
  if(baseRange < 1e-6){
    padding = Math.max(padding, magnitude * 0.12, 1);
  }
  min -= padding;
  max += padding;
  if(min === max){
    min -= 1;
    max += 1;
  }
  return {min, max};
}

function getNationalOutlook(){
  const score = S.stability + S.approval * 0.35 + S.nationalScore * 8 - S.bubbleIndex * 0.22 - Math.max(0, S.inflation - 4) * 3;
  if(score >= 95) return {title:'強気拡張', type:'good', body:'経済・支持率・治安が揃っており、宇宙投資や大型外交を進めやすい局面です。'};
  if(score >= 70) return {title:'安定成長', type:'good', body:'大きな破綻要因は少なく、重点投資で一段上の成長を狙えます。'};
  if(score >= 45) return {title:'警戒運営', type:'', body:'成長余地はあるものの、資源不足や負債、バブルに注意が必要です。'};
  return {title:'危機対応', type:'bad', body:'支持率・安定度・物価のどれかが危険水準です。短期の立て直しを優先しましょう。'};
}

function getStrategicAlerts(){
  const alerts = [];
  if(S.bubbleIndex >= 45) alerts.push({type:'danger', title:'市場が過熱', text:'補助金連打でバブル指数が上がっています。日銀操作か投資抑制で崩壊前に調整したい状態です。'});
  if(getTotalDebt() >= 35000) alerts.push({type:'danger', title:'債務膨張', text:'国債残高がかなり重く、返済や成長投資で信認を支える段階に入っています。'});
  if(S.approval < 40 || S.stability < 40) alerts.push({type:'danger', title:'政権基盤が不安定', text:'支持率か安定度が低下中です。教育・外交・生活コスト対策で反発を和らげましょう。'});
  if(getShortageCount() >= 6) alerts.push({type:'warn', title:'資源ショック', text:'複数の資源が枯渇寸前です。輸入再編か地域開発を急ぐ必要があります。'});
  if(S.cyberPower < getAverageCyberTarget()) alerts.push({type:'warn', title:'サイバー劣勢', text:'平均的な貿易相手よりサイバー力が低く、輸入コストと外的リスクが不利です。'});
  if(S.space.failures > S.space.successes && S.space.failures > 1) alerts.push({type:'warn', title:'宇宙計画の失速', text:'失敗が先行しています。技術投資や特殊素材の発見後に再挑戦すると安定します。'});
  if(!alerts.length) alerts.push({type:'good', title:'重大警報なし', text:'直近の国家運営は安定しています。中長期目標の達成に資源を回せます。'});
  return alerts;
}

function getAdvisorNotes(){
  const notes = [];
  const avgRelation = getAverageRelation();
  if(resources.rareearth?.amount < resources.rareearth.max * 0.08 || resources.lithium?.amount < resources.lithium.max * 0.08){
    notes.push({type:'warn', text:'IT・防衛の主材料が細っています。中国・チリ・オーストラリアとの貿易線を厚くすると株価の土台が安定します。'});
  }
  if(S.techBudget < 8){
    notes.push({type:'danger', text:'技術予算が低めです。ゼロ近辺だとGDPとIT株が鈍化しやすく、宇宙開発やサイバー戦力も伸びません。'});
  }
  if(S.bubbleIndex > 35){
    notes.push({type:'warn', text:'市場が過熱しています。今は新しい補助金より、日銀操作や実体投資で軟着陸を狙う方が安全です。'});
  }
  if(S.cyberPower < 60){
    notes.push({type:'warn', text:'サイバー力がまだ伸びしろです。貿易の割引効果とイベント耐性を考えると、5,000億投資でも十分回収しやすいです。'});
  }
  if(S.space.level === 0 && S.techBudget >= 10){
    notes.push({type:'', text:'技術予算が高い今は衛星打ち上げの好機です。成功するとサイバーと防衛の両方が伸びます。'});
  }
  if(avgRelation < 62){
    notes.push({type:'', text:'平均友好度が低めです。外交改善は輸入コストを下げるだけでなく、安定度の底上げにも効きます。'});
  }
  if(Object.values(regions).reduce((sum, region) => sum + region.developed, 0) < 8){
    notes.push({type:'', text:'地域開発の総量がまだ少なめです。序盤は国内資源の底上げが株とGDPの両方に効きます。'});
  }
  if(getPortfolioValue() === 0){
    notes.push({type:'', text:'ポートフォリオが空です。資源確保後に強いセクターへ少額投資すると、市場変動の手応えが出やすくなります。'});
  }
  return notes.slice(0, 5);
}

function getGoalDefinitions(){
  const totalDeveloped = Object.values(regions).reduce((sum, region) => sum + region.developed, 0);
  const discovered = S.discoveredMaterials.length;
  const avgRelation = getAverageRelation();
  const ownedControl = companies.filter(company => (S.ownedStocks[company.name] || 0) / company.sharesOutstanding >= 0.51).length;
  const completedResearchCount = (S.completedResearch || []).length;
  const unlockedProductsCount = (S.unlockedProducts || []).length;
  const spaceMaterialKinds = [...new Set(S.discoveredMaterials.filter(entry => spaceBodies.some(body => body.id === entry.region)).map(entry => entry.id))].length;
  const cyberAdvantageCount = tradePartners.filter(p => S.cyberPower > p.cyberPower).length;
  const goals = [
    {id:'economicPower', title:'経済大国の再起', desc:'GDP 12,500兆、技術力52、国庫32万億を同時に満たす。', progress:clamp((clamp(S.gdp / 12500, 0, 1) + clamp(S.techLevel / 52, 0, 1) + clamp(S.money / 320000, 0, 1)) / 3, 0, 1), detail:`GDP ${S.gdp.toFixed(0)} / 12500兆 | 技術力 ${S.techLevel.toFixed(1)} / 52 | 国庫 ${Math.round(S.money)} / 320000億`, reward:'報酬: 国庫+8,000億 / 政治資本+20'},
    {id:'techSupremacy', title:'技術覇権', desc:'技術力62、サイバー125、技術予算20%以上、研究4件以上を達成。', progress:clamp((clamp(S.techLevel / 62, 0, 1) + clamp(S.cyberPower / 125, 0, 1) + clamp(S.techBudget / 20, 0, 1) + clamp(completedResearchCount / 4, 0, 1)) / 4, 0, 1), detail:`技術力 ${S.techLevel.toFixed(1)} / 62 | サイバー ${S.cyberPower.toFixed(0)} / 125 | 技術予算 ${S.techBudget}% / 20% | 研究 ${completedResearchCount}/4`, reward:'報酬: サイバー+5 / IT株一斉上昇'},
    {id:'energySecurity', title:'エネルギー安保', desc:'原発6基、エネルギーボーナス26%以上、ヘリウム3またはウラン備蓄を大規模安定運用。', progress:clamp((clamp(S.nuclearPlants / 6, 0, 1) + clamp(S.energyBonus / 26, 0, 1) + clamp(Math.max((resources.uranium?.amount || 0) / 12000, (resources.helium3?.amount || 0) / 26000), 0, 1)) / 3, 0, 1), detail:`原発 ${S.nuclearPlants}/6 | エネルギー ${S.energyBonus.toFixed(1)}%/26% | 戦略燃料大規模確保`, reward:'報酬: 支持率+4 / GDP微増'},
    {id:'resourceFrontier', title:'資源フロンティア', desc:'開発総量34以上、特殊素材8種類、宇宙資源4種類以上を確保する。', progress:clamp((clamp(totalDeveloped / 34, 0, 1) + clamp(discovered / 8, 0, 1) + clamp(spaceMaterialKinds / 4, 0, 1)) / 3, 0, 1), detail:`開発 ${totalDeveloped}/34 | 特殊素材 ${discovered}/8 | 宇宙資源 ${spaceMaterialKinds}/4`, reward:'報酬: 開発ブーム / 資源価格追い風'},
    {id:'publicTrust', title:'国民と国際信認', desc:'支持率82%以上、安定度88以上、平均友好度80以上。', progress:clamp((clamp(S.approval / 82, 0, 1) + clamp(S.stability / 88, 0, 1) + clamp(avgRelation / 80, 0, 1)) / 3, 0, 1), detail:`支持率 ${S.approval.toFixed(1)} / 82 | 安定度 ${S.stability.toFixed(0)} / 88 | 平均友好 ${avgRelation.toFixed(0)} / 80`, reward:'報酬: 政治資本+25 / 外交追い風'},
    {id:'spaceAge', title:'宇宙時代への進出', desc:'宇宙レベル6、宇宙資源320、軌道施設4基、惑星遠征成功7回。', progress:clamp((clamp(S.space.level / 6, 0, 1) + clamp(S.space.resources / 320, 0, 1) + clamp(S.space.factories / 4, 0, 1) + clamp(S.space.successes / 7, 0, 1)) / 4, 0, 1), detail:`宇宙Lv ${S.space.level}/6 | 宇宙資源 ${Math.round(S.space.resources)}/320 | 軌道施設 ${S.space.factories}/4 | 成功 ${S.space.successes}/7`, reward:'報酬: GDP成長 / 宇宙産業ボーナス'},
    {id:'cyberMastery', title:'情報覇権', desc:'サイバー攻撃成功5回、サイバー135、主要国の8割以上で優勢を取る。', progress:clamp((clamp((S.cyberSuccesses || 0) / 5, 0, 1) + clamp(S.cyberPower / 135, 0, 1) + clamp(cyberAdvantageCount / Math.max(1, Math.ceil(tradePartners.length * 0.8)), 0, 1)) / 3, 0, 1), detail:`攻撃成功 ${S.cyberSuccesses || 0}/5 | サイバー ${S.cyberPower}/135 | 優勢国 ${cyberAdvantageCount}/${Math.ceil(tradePartners.length * 0.8)}`, reward:'報酬: サイバー+6 / 貿易優位'},
    {id:'sovereignIndustry', title:'産業主権', desc:'過半数保有企業5社以上、国内開発34以上、技術力58以上、製品解放5種以上。', progress:clamp((clamp(ownedControl / 5, 0, 1) + clamp(totalDeveloped / 34, 0, 1) + clamp(S.techLevel / 58, 0, 1) + clamp(unlockedProductsCount / 5, 0, 1)) / 4, 0, 1), detail:`過半数保有 ${ownedControl}/5 | 開発 ${totalDeveloped}/34 | 技術力 ${S.techLevel.toFixed(1)}/58 | 製品 ${unlockedProductsCount}/5`, reward:'報酬: 配当ブースト / 安定度上昇'},
  ];
  goals.forEach(goal => goal.done = !!S.completedGoals[goal.id]);
  return goals;
}

function checkNationalGoals(){
  getGoalDefinitions().forEach(goal => {
    if(goal.progress < 1 || S.completedGoals[goal.id]) return;
    S.completedGoals[goal.id] = true;
    switch(goal.id){
      case 'economicPower':
        S.money += 8000;
        S.polCap += 20;
        break;
      case 'techSupremacy':
        S.cyberPower += 5;
        companies.filter(c => c.sector === 'tech').forEach(c => c.price = Math.round(c.price * 1.05));
        break;
      case 'energySecurity':
        S.approval = Math.min(100, S.approval + 4);
        S.gdp *= 1.01;
        break;
      case 'resourceFrontier':
        Object.values(resources).forEach(resource => { if(resource.max) resource.amount += resource.max * 0.04; });
        break;
      case 'publicTrust':
        S.polCap = Math.min(S.polCapMax, S.polCap + 25);
        tradePartners.forEach(partner => partner.relation = Math.min(100, partner.relation + 4));
        break;
      case 'spaceAge':
        S.space.resources += 20;
        S.gdp *= 1.012;
        break;
      case 'cyberMastery':
        S.cyberPower += 6;
        break;
      case 'sovereignIndustry':
        S.stability = Math.min(100, S.stability + 6);
        break;
    }
    addEvent(`🏆 国家目標「${goal.title}」達成`, 'good');
  });
  S.nationalScore = Object.values(S.completedGoals).filter(Boolean).length;
  if(S.nationalScore === getGoalDefinitions().length && !S.grandVictory){
    S.grandVictory = true;
    S.money += 15000;
    S.polCap = Math.min(S.polCapMax, S.polCap + 50);
    addEvent('👑 日本が総合国家プロジェクトを完遂。新たな黄金時代が始まった！', 'good');
  }
}

function getSaveStorageKey(slot){
  return `${STORAGE_PREFIX}:${slot}`;
}

function getSaveLabel(slot, meta=null){
  return (meta?.label || S.saveLabels?.[slot] || getDefaultSaveLabel(slot)).trim();
}

window.renameSaveSlot = function(slot){
  const meta = getSaveMeta(slot);
  const current = getSaveLabel(slot, meta);
  const next = window.prompt(S.language === 'en' ? 'Enter a save name' : 'セーブ名を入力してください', current);
  if(next === null) return;
  const label = next.trim() || getDefaultSaveLabel(slot);
  if(!S.saveLabels || typeof S.saveLabels !== 'object') S.saveLabels = {};
  S.saveLabels[slot] = label;
  try{
    const payload = readStoredSavePayload(slot);
    if(payload){
      payload.meta = {...(payload.meta || {}), label};
      writeStoredSavePayload(slot, payload);
    }
  } catch(error){}
  updateHomeScreen(S.language === 'en' ? `💾 Renamed to ${label}.` : `💾 ${label} に名前を変更しました。`);
  if(document.getElementById('actionModal').classList.contains('show') && document.getElementById('actionModal').dataset.action === 'save') openAction('save');
};

function trimArrayTail(list, maxItems){
  if(!Array.isArray(list)) return list;
  return list.length > maxItems ? list.slice(-maxItems) : list.slice();
}

function trimCompanyHistoryForSave(company, historyLimit=260){
  if(!company || typeof company !== 'object') return company;
  if(Array.isArray(company.hist)) company.hist = trimArrayTail(company.hist, historyLimit);
  if(Array.isArray(company.revenueHist)) company.revenueHist = trimArrayTail(company.revenueHist, historyLimit);
  if(Array.isArray(company.demandHist)) company.demandHist = trimArrayTail(company.demandHist, historyLimit);
  return company;
}

function buildCompactSaveState(historyLimit=520){
  const state = JSON.parse(JSON.stringify(S));
  if(state.history && typeof state.history === 'object'){
    Object.keys(state.history).forEach(key => {
      if(Array.isArray(state.history[key])) state.history[key] = trimArrayTail(state.history[key], historyLimit);
    });
  }
  state.eventHistory = trimArrayTail(state.eventHistory, 160) || [];
  state.worldLog = trimArrayTail(state.worldLog, 48) || [];
  state.mapEffects = [];
  state.pendingActions = [];
  state.tradeShipments = trimArrayTail(state.tradeShipments, 48) || [];
  state.spyMissions = trimArrayTail(state.spyMissions, 24) || [];
  state.militaryProjects = trimArrayTail(state.militaryProjects, 32) || [];
  state.cyberMiniGame = null;
  state.draggingWork = null;
  state.draggingRoute = null;
  state.draggingStrike = null;
  state.draggingDisaster = null;
  state.draggingWarDispatch = null;
  state.draggingNuke = null;
  state.mapDragging = false;
  state.mapHoverTarget = null;
  state.turnProcessing = false;
  state.focusTrayOpen = false;
  state.mobileTopCollapsed = false;
  state.mobileBottomCollapsed = false;
  state.mobilePanelCollapsed = false;
  state.mobileMapHudCollapsed = false;
  if(Array.isArray(state.foreignCompanies)){
    state.foreignCompanies = state.foreignCompanies.map(company => trimCompanyHistoryForSave(company, 220));
  }
  return state;
}

function buildCompactSaveResources(priceHistoryLimit=260){
  const data = JSON.parse(JSON.stringify(resources));
  Object.values(data).forEach(resource => {
    if(Array.isArray(resource.priceHist)) resource.priceHist = trimArrayTail(resource.priceHist, priceHistoryLimit);
  });
  return data;
}

function buildCompactSaveCompanies(historyLimit=260){
  return JSON.parse(JSON.stringify(companies)).map(company => trimCompanyHistoryForSave(company, historyLimit));
}

function createSaveData(slot, options={}){
  const label = getSaveLabel(slot);
  const historyLimit = options.compactLevel >= 2 ? 220 : options.compactLevel >= 1 ? 320 : 520;
  const companyHistoryLimit = options.compactLevel >= 2 ? 180 : options.compactLevel >= 1 ? 220 : 260;
  const resourceHistoryLimit = options.compactLevel >= 2 ? 180 : options.compactLevel >= 1 ? 220 : 260;
  return {
    version:S.saveVersion,
    savedAt:new Date().toISOString(),
    meta:{slot, label, year:S.year, week:S.week, money:S.money, gdp:S.gdp, approval:S.approval, score:S.nationalScore},
    state:buildCompactSaveState(historyLimit),
    resources:buildCompactSaveResources(resourceHistoryLimit),
    companies:buildCompactSaveCompanies(companyHistoryLimit),
    regions:JSON.parse(JSON.stringify(regions)),
    tradePartners:JSON.parse(JSON.stringify(tradePartners)),
  };
}
function getStoredSaveMeta(slot){
  const raw = localStorage.getItem(getSaveStorageKey(slot));
  if(!raw) return null;
  const decoded = decodeProtectedSaveEnvelope(raw, slot);
  const meta = decoded.meta || decoded.payload?.meta;
  if(!meta) return null;
  return {
    ...meta,
    savedAt:decoded.payload?.savedAt || decoded.meta?.savedAt || decoded.payload?.meta?.savedAt || JSON.parse(raw)?.savedAt || '',
    protected:!!decoded.protected,
    locked:!!decoded.installationMismatch || !decoded.valid,
  };
}

function syncObject(target, source){
  Object.keys(target).forEach(key => delete target[key]);
  Object.entries(source).forEach(([key, value]) => { target[key] = value; });
}

function syncArray(target, source){
  target.length = 0;
  source.forEach(item => target.push(item));
}

function applySaveData(saveData){
  const preservedLanguage = getPreferredLanguage(S.language);
  invalidateRuntimeCaches('apply-save');
  Object.keys(S).forEach(key => delete S[key]);
  Object.assign(S, saveData.state || {});
  S.language = getPreferredLanguage(preservedLanguage || S.language);
  syncObject(resources, saveData.resources || {});
  syncArray(companies, saveData.companies || []);
  syncObject(regions, saveData.regions || {});
  syncArray(tradePartners, saveData.tradePartners || []);
  ensureStateShape();
  S.mapDragging = false;
  S.mapDragStart = {x:0,y:0};
  companies.forEach(company => {
    if(!Array.isArray(company.hist)) company.hist = [company.price];
    if(typeof company.base !== 'number') company.base = company.price;
    if(typeof company.prevPrice !== 'number') company.prevPrice = company.price;
    if(typeof company.sharesOutstanding !== 'number') company.sharesOutstanding = company.price >= 20000 ? 5000 : company.price >= 5000 ? 2500 : 1200;
    if(typeof company.revenue !== 'number') company.revenue = Math.round(company.price * company.sharesOutstanding * rand(0.05, 0.14));
    if(typeof company.prevRevenue !== 'number') company.prevRevenue = company.revenue;
    if(typeof company.margin !== 'number') company.margin = rand(0.05,0.18);
    if(typeof company.volatility !== 'number') company.volatility = rand(0.004, 0.016);
    if(typeof company.sentiment !== 'number') company.sentiment = rand(-0.01,0.01);
    if(!Array.isArray(company.revenueHist)) company.revenueHist = [company.revenue];
  });
  ensureCompanySystems();
  ensureCompanyMapPositions();
  if(!Array.isArray(S.publicWorks)) S.publicWorks = [];
  if(!S.publicWorks.length) seedInitialPublicWorks();
  normalizeSeededPublicWorks();
  Object.values(resources).forEach(resource => {
    if(resource.price && !Array.isArray(resource.priceHist)) resource.priceHist = [resource.price];
    if(resource.price && typeof resource.basePrice !== 'number') resource.basePrice = resource.price;
  });
  Object.values(regions).forEach(region => { if(!region.disaster) region.disaster = null; });
  tradePartners.forEach((partner, idx) => {
    if(typeof partner.gdp !== 'number') partner.gdp = 1800 + partner.economy * 55;
    if(typeof partner.techLevel !== 'number') partner.techLevel = 20 + partner.cyberPower * 0.45;
    if(typeof partner.stability !== 'number') partner.stability = 48 + partner.relation * 0.25;
    if(typeof partner.health !== 'number') partner.health = 52 + ((idx * 7) % 28);
  });
  S.trackedPrices = S.trackedPrices.filter(key => resources[key]?.price);
  if(!S.trackedPrices.length) S.trackedPrices = ['copper','oil','gold','lithium','silicon'].filter(key => resources[key]);
  loadViewForMode(S.currentMapMode);
  pushHistoryPoint(true);
  updateChromeLanguage();
  auditEconomyIntegrity('apply-save');
  updateUI();
  updateHomeScreen();
}

function getSaveMeta(slot){
  try{
    return getStoredSaveMeta(slot);
  } catch(error){
    return null;
  }
}

function setLastResumeSlot(slot){
  if(!SAVE_SLOTS.includes(slot)) return;
  try{
    localStorage.setItem(LAST_RESUME_SLOT_KEY, slot);
  } catch(error){}
}

function getStoredResumeSlot(){
  try{
    const slot = localStorage.getItem(LAST_RESUME_SLOT_KEY);
    const meta = slot && SAVE_SLOTS.includes(slot) ? getSaveMeta(slot) : null;
    if(slot && meta && !meta.locked) return slot;
  } catch(error){}
  return '';
}

function getMostRecentSaveSlot(){
  const preferredSlot = getStoredResumeSlot();
  let bestSlot = preferredSlot && getSaveMeta(preferredSlot) ? preferredSlot : '';
  let bestTime = bestSlot ? (Date.parse(getSaveMeta(bestSlot)?.savedAt) || 0) : 0;
  SAVE_SLOTS.forEach(slot => {
    const meta = getSaveMeta(slot);
    if(!meta?.savedAt || meta.locked) return;
    const savedAt = Date.parse(meta.savedAt) || 0;
    if(savedAt > bestTime || (!bestSlot && savedAt >= bestTime) || (slot === preferredSlot && savedAt === bestTime)){
      bestTime = savedAt;
      bestSlot = slot;
    }
  });
  const autoMeta = getSaveMeta('autosave');
  return bestSlot || (autoMeta && !autoMeta.locked ? 'autosave' : '');
}

function autoSaveForRelog(){
  if(!S.started || !S.autoSaveEnabled) return false;
  const ok = performSave('autosave', {silent:true});
  if(ok){
    S.lastAutoSaveWeek = S.totalWeeks || S.lastAutoSaveWeek || 0;
  }
  return ok;
}

function performSave(slot, options={}){
  try{
    verifyRuntimeIntegrity('save');
    auditEconomyIntegrity('save');
    S.lastUsedSaveSlot = slot;
    if(!options.preserveResumeSlot) setLastResumeSlot(slot);
    let saved = false;
    let lastError = null;
    [0, 1, 2].some(compactLevel => {
      try{
        const payload = createSaveData(slot, {compactLevel});
        writeStoredSavePayload(slot, payload);
        saved = true;
        return true;
      } catch(error){
        lastError = error;
        return false;
      }
    });
    if(!saved) throw lastError || new Error('save failed');
    if(!options.silent) addEvent(S.language === 'en' ? `💾 Saved to ${getSaveLabel(slot)}` : `💾 ${getSaveLabel(slot)} に保存`, 'good');
    if(document.getElementById('actionModal').classList.contains('show') && document.getElementById('actionModal').dataset.action === 'save') openAction('save');
    updateHomeScreen(S.language === 'en' ? `💾 Saved ${getSaveLabel(slot)}.` : `💾 ${getSaveLabel(slot)} を保存しました。`);
    return true;
  } catch(error){
    if(!options.silent) addEvent(S.language === 'en' ? '❌ Save failed: check browser storage.' : '❌ セーブ失敗: ブラウザ保存領域を確認してください', 'bad');
    return false;
  }
}

window.saveGame = function(slot){ performSave(slot); };

window.loadGame = function(slot){
  try{
    verifyRuntimeIntegrity('load');
    const parsed = readStoredSavePayload(slot);
    if(!parsed){ addEvent(S.language === 'en' ? '❌ No save data found.' : '❌ セーブデータがありません', 'bad'); return; }
    applySaveData(parsed);
    S.lastUsedSaveSlot = slot;
    setLastResumeSlot(slot);
    if(parsed?.meta?.label) S.saveLabels[slot] = parsed.meta.label;
    addEvent(S.language === 'en' ? `📂 Loaded ${getSaveLabel(slot, parsed?.meta)}` : `📂 ${getSaveLabel(slot, parsed?.meta)} を読込`, 'good');
    closeModal();
  } catch(error){
    const message = /another installation/i.test(String(error?.message || ''))
      ? (S.language === 'en' ? '❌ Load blocked: this protected save belongs to another installation.' : '❌ ロード拒否: この保護セーブは別端末のデータです。')
      : (S.language === 'en' ? '❌ Load failed: save data may be damaged.' : '❌ ロード失敗: データ破損の可能性があります');
    addEvent(message, 'bad');
  }
};

window.deleteSave = function(slot){
  localStorage.removeItem(getSaveStorageKey(slot));
  if(S.lastUsedSaveSlot === slot) S.lastUsedSaveSlot = 'autosave';
  if(getStoredResumeSlot() === slot){
    try{ localStorage.removeItem(LAST_RESUME_SLOT_KEY); } catch(error){}
  }
  addEvent(S.language === 'en' ? `🗑 Deleted ${slot.toUpperCase()}` : `🗑 ${slot.toUpperCase()} を削除`, '');
  openAction('save');
  updateHomeScreen(S.language === 'en' ? 'Deleted the save data.' : 'セーブデータを削除しました。');
};

function maybeAutoSave(){
  if(!S.autoSaveEnabled || S.totalWeeks === 0) return;
  if(S.totalWeeks - S.lastAutoSaveWeek < 13) return;
  S.lastAutoSaveWeek = S.totalWeeks;
  performSave('autosave', {silent:true});
}

window.exportSaveData = function(){
  const payload = createSaveData('export');
  const blob = new Blob([encodeProtectedSaveEnvelope(payload, 'export')], {type:'application/json'});
  const url = URL.createObjectURL(blob);
  const anchor = document.createElement('a');
  anchor.href = url;
  anchor.download = `japan-econ-sim-${S.year}-w${S.week}.json`;
  anchor.click();
  URL.revokeObjectURL(url);
  addEvent('💾 セーブデータを書き出し', 'good');
};

function renderSaveManager(){
  const t = currentI18n();
  const saveText = t.saveManager || {};
  const slots = SAVE_SLOTS.map(slot => ({id:slot, label:slot === 'autosave' ? 'AUTO' : slot.toUpperCase(), loadOnly:slot === 'autosave'}));
  let html = `<div class="section"><h3>${saveText.title || '💾 Save Manager'}</h3><p style="font-size:9px;color:#888;line-height:1.5">${saveText.desc || ''}</p>`;
  slots.forEach(slot => {
    const meta = getSaveMeta(slot.id);
    const label = getSaveLabel(slot.id, meta);
    const locked = !!meta?.locked;
    html += `<div class="save-slot">
      <div class="slot-title"><span>${slot.label}</span><span class="pill">${meta ? new Date(meta.savedAt).toLocaleString(S.language === 'en' ? 'en-US' : 'ja-JP') : (saveText.slotEmpty || 'Empty')}</span></div>
      <div class="save-slot-name">${label}</div>
      <div class="slot-meta">${meta ? `${saveText.yearWeek || 'Date'}: ${formatWeekText(meta.year, meta.week)}<br>${saveText.treasury || 'Treasury'}: ${formatMoneyCompact(meta.money)}<br>GDP: ${formatGdpCompact(meta.gdp, 1)} | ${saveText.approval || 'Approval'}: ${meta.approval.toFixed(1)}% | ${saveText.score || 'Goals'}: ${meta.score}${locked ? `<br>${S.language === 'en' ? 'Protected save: different installation' : '保護セーブ: 別端末データ'}` : ''}` : (saveText.empty || 'No save data in this slot yet.')}</div>
      <div class="save-actions">
        ${slot.loadOnly ? '' : `<button class="btn btn-green" onclick="saveGame('${slot.id}')">${saveText.save || 'Save'}</button>`}
        <button class="btn btn-blue" onclick="loadGame('${slot.id}')" ${!meta || locked ? 'disabled' : ''}>${saveText.load || 'Load'}</button>
        ${slot.loadOnly ? '' : `<button class="btn btn-sm" onclick="renameSaveSlot('${slot.id}')">${saveText.rename || 'Rename'}</button>`}
        ${slot.loadOnly ? '' : `<button class="btn" onclick="deleteSave('${slot.id}')" ${!meta ? 'disabled' : ''}>${saveText.delete || 'Delete'}</button>`}
      </div>
    </div>`;
  });
  html += `<div class="save-actions">
    <button class="btn btn-yellow" onclick="exportSaveData()">${saveText.export || 'Export JSON'}</button>
    <button class="btn btn-blue" onclick="beginWorldCreation()">${S.language === 'en' ? '🧭 New World Setup' : '🧭 新しいワールド'}</button>
    <span class="pill">${S.language === 'en' ? `${getDifficultyMeta().labelEn} / ${S.sandboxMode ? 'Sandbox' : 'Standard'}` : `${getDifficultyMeta().labelJa} / ${S.sandboxMode ? '試験' : '通常'}`}</span>
  </div>
  <p class="compact-note" style="margin-top:6px">${saveText.note || ''}</p></div>`;
  return html;
}

let WORLD_CREATION_DRAFT = {difficultyMode:'normal', sandboxMode:false};

function resetWorldCreationDraft(){
  WORLD_CREATION_DRAFT = {difficultyMode:'normal', sandboxMode:false};
}

window.beginWorldCreation = function(){
  resetWorldCreationDraft();
  openAction('worldsetup');
};

function getWorldConfigLabel(mode='normal', sandbox=false, lang=S.language){
  const difficultyMeta = getDifficultyMeta(mode);
  if(lang === 'en') return `${difficultyMeta.labelEn} / ${sandbox ? 'Sandbox' : 'Standard'}`;
  return `${difficultyMeta.labelJa} / ${sandbox ? '試験' : '通常'}`;
}

function renderWorldCreationPanel(){
  const isEn = S.language === 'en';
  const difficultyMeta = getDifficultyMeta(WORLD_CREATION_DRAFT.difficultyMode);
  const summaryLabel = getWorldConfigLabel(WORLD_CREATION_DRAFT.difficultyMode, WORLD_CREATION_DRAFT.sandboxMode, S.language);
  const startMoney = difficultyMeta.startMoney || DEFAULT_STATE.money;
  const polCapCap = WORLD_CREATION_DRAFT.sandboxMode ? 6000 : 400;
  return `<div class="section">
    <h3>${isEn ? '🧭 World Creation' : '🧭 ワールド作成'}</h3>
    <p class="compact-note">${isEn
      ? 'Choose the difficulty and sandbox rules for the next world. Once the world is created, these rules stay fixed for that campaign.'
      : '次に作るワールドの難易度と試験モードをここで決めます。ワールド作成後、このルールはそのキャンペーン中ずっと固定されます。'}</p>
    <div class="headline-card" style="margin-bottom:10px">
      <div class="meta">${isEn ? 'Current Selection' : '現在の選択'}</div>
      <div class="body"><strong>${summaryLabel}</strong><br>${isEn ? 'Opening treasury' : '初期資金'}: ${formatMoneyCompact(startMoney)} / ${isEn ? 'Political cap' : '政治資本上限'}: ${polCapCap}</div>
    </div>
    <div class="section" style="margin-top:0">
      <h3>${isEn ? 'Difficulty' : '難易度'}</h3>
      <div class="save-actions">
        <button class="btn btn-sm ${WORLD_CREATION_DRAFT.difficultyMode==='easy'?'active':''}" onclick="setWorldCreationDifficulty('easy')">EASY</button>
        <button class="btn btn-sm ${WORLD_CREATION_DRAFT.difficultyMode==='normal'?'btn-blue active':''}" onclick="setWorldCreationDifficulty('normal')">NORMAL</button>
        <button class="btn btn-sm ${WORLD_CREATION_DRAFT.difficultyMode==='hard'?'btn-yellow active':''}" onclick="setWorldCreationDifficulty('hard')">HARD</button>
      </div>
      <div class="compact-note" style="margin-top:6px">${isEn ? difficultyMeta.noteEn : difficultyMeta.noteJa}</div>
    </div>
    <div class="section">
      <h3>${isEn ? 'Rule Set' : 'ルールセット'}</h3>
      <div class="save-actions">
        <button class="btn btn-sm ${!WORLD_CREATION_DRAFT.sandboxMode?'btn-blue active':''}" onclick="setWorldCreationSandbox(false)">${isEn ? 'Standard World' : '通常ワールド'}</button>
        <button class="btn btn-sm ${WORLD_CREATION_DRAFT.sandboxMode?'btn-green active':''}" onclick="setWorldCreationSandbox(true)">${isEn ? 'Sandbox World' : '試験ワールド'}</button>
      </div>
      <div class="compact-note" style="margin-top:6px">${WORLD_CREATION_DRAFT.sandboxMode
        ? (isEn ? 'Sandbox worlds start with abundant funds and political capital, but that rule cannot be turned off later inside this world.' : '試験ワールドは高い資金と政治資本で始まりますが、このワールドの中では後から通常ルールへ変更できません。')
        : (isEn ? 'Standard worlds use the normal approval and political-cap rules. To change this later, create a different world.' : '通常ワールドは通常の支持率・政治資本ルールで進みます。後から変えたい場合は別のワールドを作ります。')}</div>
    </div>
    <div class="detail-grid" style="margin-top:10px">
      <div class="dg-item"><div class="dg-label">${isEn ? 'Opening Money' : '初期資金'}</div><div class="dg-value">${formatMoneyCompact(startMoney)}</div></div>
      <div class="dg-item"><div class="dg-label">${isEn ? 'Political Cap' : '政治資本上限'}</div><div class="dg-value">${polCapCap}</div></div>
      <div class="dg-item"><div class="dg-label">${isEn ? 'Rule Lock' : 'ルール固定'}</div><div class="dg-value">${isEn ? 'Locked after creation' : '作成後は固定'}</div></div>
    </div>
    <div class="save-actions" style="margin-top:12px">
      <button class="btn btn-green" onclick="confirmWorldCreation()">${isEn ? 'Create World' : 'この設定で開始'}</button>
      <button class="btn" onclick="closeModal()">${isEn ? 'Cancel' : 'キャンセル'}</button>
    </div>
  </div>`;
}

function renderSettingsPanel(){
  const t = currentI18n().settings || {};
  const autoMeta = getSaveMeta('autosave');
  const isEn = S.language === 'en';
  const focusStats = getFocusTrayStatCatalog();
  const activation = getActivationInfo();
  const security = readSecurityState();
  const installShort = getInstallationId().slice(-12);
  return `<div class="section"><h3>${t.title || '⚙ Settings'}</h3>
    <p class="compact-note">${t.desc || ''}</p>
    <div class="home-setting-row">
      <span>${t.language || 'Language'}</span>
      <div class="save-actions">
        <button class="btn btn-sm ${S.language==='ja'?'active':''}" onclick="setLanguage('ja');openAction('settings')">日本語</button>
        <button class="btn btn-sm ${S.language==='en'?'active':''}" onclick="setLanguage('en');openAction('settings')">English</button>
      </div>
    </div>
    <div class="compact-note" style="margin-bottom:10px">${t.languageNote || ''}</div>
    <div class="home-setting-row">
      <span>${t.autosave || 'Autosave'}</span>
      <button class="btn btn-sm ${S.autoSaveEnabled ? 'btn-blue active' : ''}" onclick="toggleAutoSaveFromHome();openAction('settings')">${S.autoSaveEnabled ? 'ON' : 'OFF'}</button>
    </div>
    <div class="compact-note" style="margin-bottom:10px">${S.autoSaveEnabled ? (t.autosaveOn || '') : (t.autosaveOff || '')}</div>
    <div class="home-setting-row">
      <span>${t.sandbox || 'Sandbox Mode'}</span>
      <span class="pill">${getWorldConfigLabel(S.difficultyMode, S.sandboxMode, S.language)}</span>
    </div>
    <div class="compact-note" style="margin-bottom:10px">${isEn ? 'World rules are fixed after creation. Use the world creation screen when you want a different difficulty or sandbox setup.' : 'ワールド作成後は難易度と試験モードが固定されます。別の設定で遊ぶ時は新しいワールドを作成してください。'}</div>
    <div class="save-actions" style="margin-bottom:10px">
      <button class="btn btn-sm btn-blue" onclick="beginWorldCreation()">${isEn ? '🧭 Create New World' : '🧭 新しいワールドを作成'}</button>
    </div>
    <div class="home-setting-row">
      <span>${t.focusMode || 'Focus Mode'}</span>
      <button class="btn btn-sm ${S.focusMode ? 'btn-purple active' : ''}" onclick="toggleFocusMode()">${S.focusMode ? 'ON' : 'OFF'}</button>
    </div>
    <div class="compact-note" style="margin-bottom:10px">${S.focusMode ? (t.focusModeOn || '') : (t.focusModeOff || '')}</div>
    <div class="home-setting-row">
      <span>${isEn ? 'Focus Tray Position' : '集中トレイ位置'}</span>
      <div class="save-actions">
        <button class="btn btn-sm ${S.focusTrayPosition==='left'?'active':''}" onclick="setFocusTrayPosition('left')">${isEn ? 'Left' : '左'}</button>
        <button class="btn btn-sm ${S.focusTrayPosition==='bottom'?'active':''}" onclick="setFocusTrayPosition('bottom')">${isEn ? 'Bottom' : '下'}</button>
        <button class="btn btn-sm ${S.focusTrayOpen?'btn-blue active':''}" onclick="toggleFocusTray()">${S.focusTrayOpen ? (isEn ? 'Open' : '開') : (isEn ? 'Closed' : '閉')}</button>
      </div>
    </div>
    <div class="compact-note" style="margin-bottom:10px">${isEn ? 'Choose which top stats remain visible inside focus mode.' : '集中モードで持ち込む上部情報を選べます。'}</div>
    <div class="save-actions" style="margin-bottom:10px">
      ${focusStats.map(item => `<button class="btn btn-sm ${(S.focusTrayStats || []).includes(item.id)?'active':''}" onclick="toggleFocusTrayStat('${item.id}')">${item.label}</button>`).join('')}
    </div>
    <div class="home-setting-row">
      <span>${t.uiMode || (isEn ? 'Screen Mode' : '画面モード')}</span>
      <div class="save-actions">
        <button class="btn btn-sm ${S.uiMode==='desktop'?'active':''}" onclick="setUiMode('desktop');openAction('settings')">${t.uiDesktop || (isEn ? 'Desktop' : 'PC')}</button>
        <button class="btn btn-sm ${S.uiMode==='mobile'?'btn-blue active':''}" onclick="setUiMode('mobile');openAction('settings')">${t.uiMobile || (isEn ? 'Mobile' : 'スマホ')}</button>
      </div>
    </div>
    <div class="compact-note" style="margin-bottom:10px">${S.uiMode === 'mobile' ? (t.uiMobileNote || '') : (isEn ? 'Desktop mode keeps the full HUD visible.' : 'PCモードではフルHUDを表示します。')}</div>
    ${S.uiMode === 'mobile' ? `<div class="home-setting-row">
      <span>${isEn ? 'Fold Controls' : '折りたたみ'}</span>
      <div class="save-actions">
        <button class="btn btn-sm ${S.mobileTopCollapsed?'':'active'}" onclick="toggleMobileChrome('top');openAction('settings')">${t.mobileTop || (isEn ? 'Top' : '上')} ${S.mobileTopCollapsed ? (t.closed || (isEn ? 'Closed' : '閉')) : (t.opened || (isEn ? 'Open' : '開'))}</button>
        <button class="btn btn-sm ${S.mobileBottomCollapsed?'':'active'}" onclick="toggleMobileChrome('bottom');openAction('settings')">${t.mobileBottom || (isEn ? 'Bottom' : '下')} ${S.mobileBottomCollapsed ? (t.closed || (isEn ? 'Closed' : '閉')) : (t.opened || (isEn ? 'Open' : '開'))}</button>
        <button class="btn btn-sm ${S.mobilePanelCollapsed?'':'active'}" onclick="toggleMobileChrome('panel');openAction('settings')">${t.mobilePanel || (isEn ? 'Panel' : '右')} ${S.mobilePanelCollapsed ? (t.closed || (isEn ? 'Closed' : '閉')) : (t.opened || (isEn ? 'Open' : '開'))}</button>
        <button class="btn btn-sm ${S.mobileMapHudCollapsed?'':'active'}" onclick="toggleMobileChrome('maphud');openAction('settings')">${isEn ? 'Map HUD' : '地図HUD'} ${S.mobileMapHudCollapsed ? (t.closed || (isEn ? 'Closed' : '閉')) : (t.opened || (isEn ? 'Open' : '開'))}</button>
      </div>
    </div>` : ''}
    <div class="home-setting-row">
      <span>${isEn ? 'Graphics Quality' : '画質設定'}</span>
      <div class="save-actions">
        <button class="btn btn-sm ${S.qualityMode==='low'?'active':''}" onclick="setQualityMode('low');openAction('settings')">${isEn ? 'Low' : '低'}</button>
        <button class="btn btn-sm ${S.qualityMode==='balanced'?'btn-blue active':''}" onclick="setQualityMode('balanced');openAction('settings')">${isEn ? 'Balanced' : '標準'}</button>
        <button class="btn btn-sm ${S.qualityMode==='high'?'btn-green active':''}" onclick="setQualityMode('high');openAction('settings')">${isEn ? 'High' : '高'}</button>
      </div>
    </div>
    <div class="compact-note" style="margin-bottom:10px">${S.qualityMode === 'low'
      ? (isEn ? 'Low quality reduces animation and blur for weaker devices.' : '低画質はアニメーションとぼかしを減らして低性能端末向けにします。')
      : S.qualityMode === 'high'
        ? (isEn ? 'High quality enables richer animation and more frequent redraws.' : '高画質はアニメーションと描画更新を多めにします。')
        : (isEn ? 'Balanced quality keeps the default effects while staying lighter than high mode.' : '標準画質は演出と軽さのバランスを取ります。')}</div>
    <div class="home-setting-row">
      <span>${t.timeMode || 'Time Flow'}</span>
      <div class="save-actions">
        <button class="btn btn-sm ${S.timeMode==='realtime'?'btn-blue active':''}" onclick="setTimeMode('realtime');openAction('settings')">${t.realtime || 'Real-time'}</button>
        <button class="btn btn-sm ${S.timeMode==='turn'?'btn-yellow active':''}" onclick="setTimeMode('turn');openAction('settings')">${t.turn || 'Turn-based'}</button>
      </div>
    </div>
    <div class="compact-note" style="margin-bottom:10px">${S.timeMode === 'turn' ? (t.timeTurnNote || '') : (t.timeRealtimeNote || '')}</div>
    <div class="home-setting-row">
      <span>${t.turnStep || 'Advance Size'}</span>
      <div class="save-actions">
        <button class="btn btn-sm ${S.turnStepWeeks===1?'active':''}" onclick="setTurnStep(1);openAction('settings')">${t.turnStep1 || '1 week'}</button>
        <button class="btn btn-sm ${S.turnStepWeeks===4?'active':''}" onclick="setTurnStep(4);openAction('settings')">${t.turnStep4 || '4 weeks'}</button>
        <button class="btn btn-sm ${S.turnStepWeeks===13?'active':''}" onclick="setTurnStep(13);openAction('settings')">${t.turnStep13 || '13 weeks'}</button>
      </div>
    </div>
    <div class="compact-note" style="margin-bottom:10px">${S.timeMode === 'turn' ? `${t.turn || 'Turn-based'} / ${getTurnStepLabel(S.turnStepWeeks, true)}` : (S.language === 'en' ? `Current speed x${S.speed}` : `現在速度 x${S.speed}`)}</div>
    <div class="home-setting-row">
      <span>${t.current || 'Current Autosave'}</span>
      <span class="compact-note">${autoMeta ? `${getSaveLabel('autosave', autoMeta)} / ${formatWeekText(autoMeta.year, autoMeta.week)}` : (currentI18n().homeAutosaveEmpty || 'None yet')}</span>
    </div>
    <div class="section" style="margin-top:12px;padding-top:10px;border-top:1px solid rgba(255,255,255,.08)">
      <h3>${isEn ? '🔐 Security' : '🔐 セキュリティ'}</h3>
      <div class="home-setting-row">
        <span>${isEn ? 'Activation' : 'ライセンス状態'}</span>
        <span class="pill ${activation.status === 'active' ? 'good' : ''}">${activation.status === 'active' ? (isEn ? 'Activated' : '認証済み') : (isEn ? 'Trial' : '試用')}</span>
      </div>
      <div class="compact-note" style="margin-bottom:6px">${isEn ? 'Installation ID' : '端末ID'}: ${escapeHtml(installShort)}</div>
      <div class="compact-note" style="margin-bottom:10px">${isEn ? 'Protected saves and integrity checks are enabled. Full anti-copy and online license enforcement still require a real server/backend.' : '保護セーブと整合性検査は有効です。完全なコピー防止やオンライン認証は別途サーバー実装が必要です。'} </div>
      <div class="save-actions">
        <button class="btn btn-sm ${activation.status === 'active' ? 'btn-green active' : 'btn-blue'}" onclick="enterActivationKey()">${isEn ? 'Enter Key' : 'キー入力'}</button>
        <button class="btn btn-sm" onclick="clearActivationKey()">${isEn ? 'Clear' : '解除'}</button>
        <button class="btn btn-sm" onclick="runSecurityScan()">${isEn ? 'Scan' : '診断'}</button>
      </div>
      <div class="compact-note" style="margin-top:8px">${isEn ? 'Tamper state' : '改ざん状態'}: ${security.tamperDetected ? (isEn ? 'Alert' : '警告あり') : (isEn ? 'Normal' : '正常')} / ${isEn ? 'Suspicious loads' : '疑わしい読込'}: ${security.suspiciousLoads || 0}</div>
    </div>
    <div class="save-actions" style="margin-top:12px">
      <button class="btn" onclick="closeModal()">${t.close || 'Close'}</button>
    </div>
  </div>`;
}

window.enterActivationKey = function(){
  const isEn = S.language === 'en';
  const current = getActivationInfo();
  const next = window.prompt(isEn ? 'Enter your activation key' : 'アクティベーションキーを入力してください', current.key || '');
  if(next === null) return;
  const normalized = normalizeActivationKey(next);
  if(!normalized){
    addEvent(isEn ? '❌ Activation key was empty.' : '❌ アクティベーションキーが空です。', 'bad');
    openAction('settings');
    return;
  }
  if(!isActivationKeyValid(normalized)){
    addEvent(isEn ? '❌ Invalid activation key.' : '❌ アクティベーションキーが無効です。', 'bad');
    openAction('settings');
    return;
  }
  persistActivationInfo({
    status:'active',
    key:normalized,
    activatedAt:new Date().toISOString(),
    installationId:getInstallationId(),
  });
  addEvent(isEn ? '🔐 Activation applied to this installation.' : '🔐 この端末でライセンス認証を適用しました。', 'good');
  openAction('settings');
};

window.clearActivationKey = function(){
  persistActivationInfo({status:'trial', key:'', activatedAt:'', installationId:getInstallationId()});
  addEvent(S.language === 'en' ? '🔐 Activation cleared for this installation.' : '🔐 この端末のライセンス認証を解除しました。', '');
  openAction('settings');
};

window.runSecurityScan = function(){
  const ok = verifyRuntimeIntegrity('manual-scan');
  auditEconomyIntegrity('manual-scan');
  if(ok) addEvent(S.language === 'en' ? '🔐 Security scan completed. No runtime mismatch found.' : '🔐 セキュリティ診断が完了しました。実行時の不一致は見つかりませんでした。', 'good');
  openAction('settings');
};

window.toggleSandboxMode = function(){
  if(S.worldConfigLocked){
    const msg = S.language === 'en'
      ? 'Sandbox mode is fixed for this world. Create a new world to change it.'
      : '試験モードはこのワールドで固定されています。変更するには新しいワールドを作成してください。';
    updateHomeScreen(msg);
    if(document.getElementById('actionModal')?.dataset.action === 'settings') openAction('settings');
    return;
  }
  S.sandboxMode = !S.sandboxMode;
  if(S.sandboxMode){
    S.money = Math.max(S.money, 650000);
    S.polCap = Math.max(S.polCap, 2800);
    S.polCapMax = Math.max(S.polCapMax, 6000);
  } else {
    S.polCapMax = 400;
    S.polCap = Math.min(S.polCap, S.polCapMax);
  }
  updateHomeScreen(S.sandboxMode ? (S.language === 'en' ? 'Sandbox mode enabled.' : '試験モードを有効化しました。') : (S.language === 'en' ? 'Sandbox mode disabled.' : '試験モードを無効化しました。'));
  if(document.getElementById('actionModal').classList.contains('show')){
    const currentAction = document.getElementById('actionModal').dataset.action;
    if(currentAction === 'settings') openAction('settings');
    else if(currentAction === 'save') openAction('save');
  }
  updateUI();
};

function captureInitialSnapshot(){
  INITIAL_SNAPSHOT = JSON.parse(JSON.stringify(createSaveData('baseline')));
}

const HOME_AMBIENT_IMAGES = [
  'assets/military-art/Modern%20Infantry%20Wide%204.png',
  'assets/military-art/Modern%20Tank%203.png',
  'assets/military-art/Modern%20Battleship%201.png',
  'assets/military-art/drone-modern.png',
  'assets/military-art/fighter-modern.png',
  'assets/military-art/spy-modern.png',
  'assets/military-art/fighter-advanced.png',
];

function refreshHomeAmbientArt(force=false){
  const layer = document.getElementById('homeAmbientLayer');
  if(!layer) return;
  const tiles = [...layer.querySelectorAll('.home-ambient-tile')];
  if(!tiles.length) return;
  const hasImages = tiles.every(tile => !!tile.style.backgroundImage);
  if(!force && hasImages) return;
  const shuffled = [...HOME_AMBIENT_IMAGES].sort(() => Math.random() - 0.5);
  tiles.forEach((tile, index) => {
    const src = shuffled[index % shuffled.length];
    tile.style.backgroundImage = `linear-gradient(145deg, rgba(7,14,28,.84), rgba(7,14,28,.62) 42%, rgba(7,14,28,.92)), url('${src}')`;
  });
}

function titleOpenSaveList(){
  const list = document.getElementById('homeSaveList');
  if(!list) return;
  S.homeSaveExpanded = !S.homeSaveExpanded;
  updateHomeScreen(S.homeSaveExpanded ? (S.language === 'en' ? 'Opened the save list.' : 'セーブ一覧を開きました。') : (S.language === 'en' ? 'Closed the save list.' : 'セーブ一覧を閉じました。'));
  if(S.homeSaveExpanded) list.scrollIntoView({ behavior:'smooth', block:'start' });
}

function getDifficultyMeta(mode=S.difficultyMode){
  const map = {
    easy:{
      labelJa:'イージー',
      labelEn:'Easy',
      noteJa:'株価は比較的安定し、序盤の下支えが強く、イベントショックも軽めです。開始資金もかなり多めです。',
      noteEn:'Stocks stay steadier, early downside protection is stronger, event shocks are softer, and you begin with a generous treasury.',
      marketSwing:0.72,
      revenueNoise:0.78,
      priceNoise:0.74,
      startupGuard:0.12,
      eventShockImpact:0.82,
      crashBias:0.92,
      priceResponse:0.9,
      startMoney:240000,
    },
    normal:{
      labelJa:'標準',
      labelEn:'Normal',
      noteJa:'通常の国家運営向け。株価・売上・イベントの揺れは標準的です。開始資金も標準です。',
      noteEn:'Balanced for standard play. Market, revenue, and event swings stay near the default profile with a standard opening treasury.',
      marketSwing:1,
      revenueNoise:1,
      priceNoise:1,
      startupGuard:0,
      eventShockImpact:1,
      crashBias:1,
      priceResponse:1,
      startMoney:90000,
    },
    hard:{
      labelJa:'ハード',
      labelEn:'Hard',
      noteJa:'株価は不規則に動きやすく、急変やイベントショックも強く、序盤の保護も薄くなります。開始資金はやや少なめです。',
      noteEn:'Stocks move more irregularly, shocks hit harder, early-game protection is weaker, and you start with a tighter treasury.',
      marketSwing:1.42,
      revenueNoise:1.24,
      priceNoise:1.38,
      startupGuard:-0.05,
      eventShockImpact:1.18,
      crashBias:1.1,
      priceResponse:1.12,
      startMoney:65000,
    }
  };
  return map[mode] || map.normal;
}

function applyDifficultyOpeningState(target=S, mode=(target?.difficultyMode || S.difficultyMode || 'normal')){
  if(!target) return;
  const difficultyMeta = getDifficultyMeta(mode);
  if(target.sandboxMode) return;
  const startMoney = Number.isFinite(difficultyMeta.startMoney) ? difficultyMeta.startMoney : DEFAULT_STATE.money;
  target.money = startMoney;
  target.prevMoney = startMoney;
}

window.setDifficultyMode = function(mode){
  if(!['easy','normal','hard'].includes(mode)) return;
  if(S.worldConfigLocked){
    const msg = S.language === 'en'
      ? 'Difficulty is fixed for this world. Create a new world to choose another one.'
      : '難易度はこのワールドで固定されています。別の難易度にするには新しいワールドを作成してください。';
    updateHomeScreen(msg);
    if(document.getElementById('actionModal')?.classList.contains('show') && document.getElementById('actionModal').dataset.action === 'settings'){
      openAction('settings');
    }
    return;
  }
  S.difficultyMode = mode;
  if(!S.started && (S.totalWeeks || 0) === 0 && S.year === DEFAULT_STATE.year && S.week === DEFAULT_STATE.week){
    applyDifficultyOpeningState(S, mode);
    updateUI();
  }
  updateHomeScreen(S.language === 'en'
    ? `Difficulty set to ${getDifficultyMeta(mode).labelEn}.`
    : `難易度を「${getDifficultyMeta(mode).labelJa}」に変更しました。`);
  if(document.getElementById('actionModal')?.classList.contains('show') && document.getElementById('actionModal').dataset.action === 'settings'){
    openAction('settings');
  }
};

window.setWorldCreationDifficulty = function(mode){
  if(!['easy','normal','hard'].includes(mode)) return;
  WORLD_CREATION_DRAFT.difficultyMode = mode;
  if(document.getElementById('actionModal')?.dataset.action === 'worldsetup') openAction('worldsetup');
};

window.setWorldCreationSandbox = function(enabled){
  WORLD_CREATION_DRAFT.sandboxMode = !!enabled;
  if(document.getElementById('actionModal')?.dataset.action === 'worldsetup') openAction('worldsetup');
};

window.confirmWorldCreation = function(){
  const config = {
    sandboxMode:!!WORLD_CREATION_DRAFT.sandboxMode,
    difficultyMode:['easy','normal','hard'].includes(WORLD_CREATION_DRAFT.difficultyMode) ? WORLD_CREATION_DRAFT.difficultyMode : 'normal',
  };
  runWithViewLoader(S.language === 'en' ? 'Creating a new world...' : '新しいワールドを作成中...', () => {
    closeModal();
    resetGameState(
      S.language === 'en'
        ? `Started a new ${getWorldConfigLabel(config.difficultyMode, config.sandboxMode, 'en')} world.`
        : `「${getWorldConfigLabel(config.difficultyMode, config.sandboxMode, 'ja')}」で新しいワールドを開始しました。`,
      true,
      config
    );
    schedulePanelRender();
    scheduleMapDraw();
    scheduleMapHudRefresh();
  });
};


function updateHomeScreen(message=S.homeMessage){
  const screen = document.getElementById('homeScreen');
  if(!screen) return;
  const t = currentI18n();
  const difficultyMeta = getDifficultyMeta();
  S.homeMessage = message || S.homeMessage || (S.language === 'en' ? 'You can start from the title screen.' : 'ホーム画面から開始できます。');
  const sandboxBtn = document.getElementById('homeSandboxBtn');
  const sandboxValueEl = document.getElementById('homeSandboxValue');
  const autosaveBtn = document.getElementById('homeAutosaveBtn');
  const autosaveMeta = document.getElementById('homeAutosaveMeta');
  const messageEl = document.getElementById('homeMessage');
  const homeSaveList = document.getElementById('homeSaveList');
  const titleEl = document.getElementById('homeTitleText');
  const subEl = document.getElementById('homeSubText');
  const infoListEl = document.getElementById('homeInfoList');
  const heroKickerEl = document.getElementById('homeHeroKicker');
  const heroValueEl = document.getElementById('homeHeroValue');
  const heroNoteEl = document.getElementById('homeHeroNote');
  const heroSaveLabelEl = document.getElementById('homeHeroSaveLabel');
  const heroSaveValueEl = document.getElementById('homeHeroSaveValue');
  const heroSaveNoteEl = document.getElementById('homeHeroSaveNote');
  const heroModeLabelEl = document.getElementById('homeHeroModeLabel');
  const heroModeValueEl = document.getElementById('homeHeroModeValue');
  const heroModeNoteEl = document.getElementById('homeHeroModeNote');
  const heroScaleLabelEl = document.getElementById('homeHeroScaleLabel');
  const heroScaleValueEl = document.getElementById('homeHeroScaleValue');
  const heroScaleNoteEl = document.getElementById('homeHeroScaleNote');
  const heroStartBtnEl = document.getElementById('homeHeroStartBtn');
  const heroContinueBtnEl = document.getElementById('homeHeroContinueBtn');
  const heroSaveBtnEl = document.getElementById('homeHeroSaveBtn');
  const heroResetBtnEl = document.getElementById('homeHeroResetBtn');
  const heroTutorialBtnEl = document.getElementById('homeHeroTutorialBtn');
  const homeDifficultyLabelEl = document.getElementById('homeDifficultyLabel');
  const homeDifficultyValueEl = document.getElementById('homeDifficultyValue');
  const homeDifficultyNoteEl = document.getElementById('homeDifficultyNote');
  const homeDifficultySettingLabelEl = document.getElementById('homeDifficultySettingLabel');
  const homeDifficultySettingNoteEl = document.getElementById('homeDifficultySettingNote');
  const homeWorldCreateBtn = document.getElementById('homeWorldCreateBtn');
  const homeSettingWorldCreateBtn = document.getElementById('homeSettingWorldCreateBtn');
  const homeSettingsOpenBtn = document.getElementById('homeSettingsOpenBtn');
  const homeExitBtn = document.getElementById('homeExitBtn');
  const homeDropSignsBtn = document.getElementById('homeDropSignsBtn');
  const ids = {
    homeBadgeMain:t.badgeMain,
    homeBadgeSetting:t.badgeSetting,
    homeStartBtn:t.start,
    homeAutoBtn:t.continue,
    homeSaveBtn:S.homeSaveExpanded ? t.homeToggleSaveClose : t.homeToggleSaveOpen,
    homeResetBtn:t.reset,
    homeSandboxLabel:t.sandbox,
    homeAutosaveLabel:t.autosave,
    homeCurrentSaveLabel:t.currentSave,
    homePresetLabel:t.preset,
    homePresetValue:t.presetValue,
  };
  if(titleEl) titleEl.textContent = t.homeTitle;
  if(subEl) subEl.textContent = t.homeSub;
  refreshHomeAmbientArt(false);
  Object.entries(ids).forEach(([id, value]) => {
    const el = document.getElementById(id);
    if(el) el.textContent = value;
  });
  if(heroStartBtnEl) heroStartBtnEl.textContent = t.start;
  if(heroContinueBtnEl){
    const slot = getMostRecentSaveSlot();
    const meta = getSaveMeta(slot);
    heroContinueBtnEl.textContent = meta ? `${t.homeContinuePrefix} ${getSaveLabel(slot, meta)}` : t.homeContinueMissing;
  }
  if(heroSaveBtnEl) heroSaveBtnEl.textContent = S.language === 'en' ? 'SAVE DATA' : 'セーブ一覧';
  if(heroResetBtnEl) heroResetBtnEl.textContent = t.reset;
  if(heroTutorialBtnEl) heroTutorialBtnEl.textContent = S.language === 'en' ? '📘 TUTORIAL' : '📘 チュートリアル';
  if(homeSettingsOpenBtn) homeSettingsOpenBtn.textContent = S.language === 'en' ? 'Settings' : '設定';
  if(homeExitBtn) homeExitBtn.textContent = S.language === 'en' ? 'Exit' : '終了';
  if(homeDropSignsBtn) homeDropSignsBtn.textContent = S.language === 'en' ? 'Drop Signs' : '看板を落とす';
  if(infoListEl) infoListEl.innerHTML = t.info.map(line => `<div>${line}</div>`).join('');
  if(sandboxBtn){
    sandboxBtn.textContent = S.sandboxMode ? 'ON' : 'OFF';
    sandboxBtn.className = `btn btn-sm ${S.sandboxMode ? 'btn-green active' : ''}`;
  }
  if(sandboxValueEl) sandboxValueEl.textContent = S.sandboxMode ? 'ON' : 'OFF';
  if(autosaveBtn){
    autosaveBtn.textContent = S.autoSaveEnabled ? 'ON' : 'OFF';
    autosaveBtn.className = `btn btn-sm ${S.autoSaveEnabled ? 'btn-blue active' : ''}`;
  }
  const autoMeta = getSaveMeta('autosave');
  if(autosaveMeta) autosaveMeta.textContent = autoMeta ? `${getSaveLabel('autosave', autoMeta)} / ${formatWeekText(autoMeta.year, autoMeta.week)}` : t.homeAutosaveEmpty;
  if(messageEl) messageEl.textContent = S.homeMessage;
  const langJaBtn = document.getElementById('langJaBtn');
  const langEnBtn = document.getElementById('langEnBtn');
  const homeUiDesktopBtn = document.getElementById('homeUiDesktopBtn');
  const homeUiMobileBtn = document.getElementById('homeUiMobileBtn');
  if(langJaBtn) langJaBtn.className = `btn btn-sm ${S.language==='ja'?'active':''}`;
  if(langEnBtn) langEnBtn.className = `btn btn-sm ${S.language==='en'?'active':''}`;
  if(homeUiDesktopBtn) homeUiDesktopBtn.className = `btn btn-sm ${S.uiMode==='desktop'?'active':''}`;
  if(homeUiMobileBtn) homeUiMobileBtn.className = `btn btn-sm ${S.uiMode==='mobile'?'btn-blue active':''}`;
  if(homeSaveList){
    homeSaveList.classList.toggle('collapsed', !S.homeSaveExpanded);
    const slots = SAVE_SLOTS.map(slot => ({id:slot, label:slot === 'autosave' ? 'AUTO' : slot.toUpperCase()}));
    homeSaveList.innerHTML = slots.map(slot => {
      const saveMeta = getSaveMeta(slot.id);
      if(!saveMeta) return '';
      const label = getSaveLabel(slot.id, saveMeta);
      return `<div class="home-save-item"><div><div class="save-slot-name">${label}</div><div class="compact-note">${slot.label} / ${formatWeekText(saveMeta.year, saveMeta.week)} / GDP ${formatGdpCompact(saveMeta.gdp, 1)}${saveMeta.locked ? ` / ${S.language === 'en' ? 'locked' : '保護'}` : ''}</div></div><div class="save-actions"><button class="btn btn-sm btn-blue" onclick="loadGame('${slot.id}');closeHomeScreen()" ${saveMeta.locked ? 'disabled' : ''}>${t.load}</button>${slot.id !== 'autosave' ? `<button class="btn btn-sm" onclick="renameSaveSlot('${slot.id}')">${S.language === 'en' ? 'Rename' : '名前'}</button>` : ''}</div></div>`;
    }).join('') || `<div class="compact-note">${t.noSave}</div>`;
  }
  const recentSlot = getMostRecentSaveSlot();
  const recentMeta = getSaveMeta(recentSlot);
  const companyCount = Array.isArray(companies) ? companies.length : 0;
  const countryCount = Array.isArray(tradePartners) ? tradePartners.length : 0;
  const prefectureCount = Array.isArray(prefectures) ? prefectures.length : 0;
  if(heroKickerEl) heroKickerEl.textContent = S.language === 'en' ? 'National Command Deck' : 'National Command Deck';
  if(heroValueEl) heroValueEl.textContent = S.language === 'en'
    ? `${prefectureCount} prefectures / ${companyCount} companies / ${countryCount} nations`
    : `${prefectureCount}都道府県 / ${companyCount}企業 / ${countryCount}か国`;
  if(heroNoteEl) heroNoteEl.textContent = S.language === 'en'
    ? 'Run policy, markets, logistics, diplomacy, war, disasters, and space in one long-form national strategy sandbox.'
    : '政策、株式市場、物流、外交、戦争、災害、宇宙を一つの国家運営にまとめた長期戦略シミュレーションです。';
  if(heroSaveLabelEl) heroSaveLabelEl.textContent = S.language === 'en' ? 'Recent Save' : 'Recent Save';
  if(heroSaveValueEl) heroSaveValueEl.textContent = recentMeta
    ? `${getSaveLabel(recentSlot, recentMeta)} / ${formatWeekText(recentMeta.year, recentMeta.week)}`
    : (S.language === 'en' ? 'No save yet' : 'まだセーブなし');
  if(heroSaveNoteEl) heroSaveNoteEl.textContent = S.language === 'en'
    ? 'Jump back into the newest campaign from the center card or save list.'
    : '中央の続きから、または保存一覧から最新の進行へすぐ戻れます。';
  if(heroModeLabelEl) heroModeLabelEl.textContent = S.language === 'en' ? 'World Rules' : 'ワールドルール';
  if(heroModeValueEl) heroModeValueEl.textContent = S.language === 'en'
    ? `English / ${S.uiMode === 'mobile' ? 'Mobile' : 'PC'} / ${difficultyMeta.labelEn} / ${S.sandboxMode ? 'Sandbox' : 'Standard'}`
    : `日本語 / ${S.uiMode === 'mobile' ? 'スマホ' : 'PC'} / ${difficultyMeta.labelJa} / ${S.sandboxMode ? '試験' : '通常'}`;
  if(heroModeNoteEl) heroModeNoteEl.textContent = S.language === 'en'
    ? 'World difficulty and sandbox rules are locked after creation. Use NEW GAME to build a different world.'
    : '難易度と試験モードはワールド作成時に固定されます。別の設定で遊ぶ時は NEW GAME から新しいワールドを作成します。';
  if(heroScaleLabelEl) heroScaleLabelEl.textContent = S.language === 'en' ? 'Play Scope' : 'Play Scope';
  if(heroScaleValueEl) heroScaleValueEl.textContent = S.language === 'en'
    ? 'Markets / Diplomacy / War / Space'
    : '市場 / 外交 / 戦争 / 宇宙';
  if(heroScaleNoteEl) heroScaleNoteEl.textContent = S.language === 'en'
    ? 'The center band is enlarged so your current scale, mode, and save status are readable at a glance.'
    : 'タイトル中央を広くして、今の規模・モード・続き状況が一目で分かるようにしています。';
  if(homeDifficultyLabelEl) homeDifficultyLabelEl.textContent = S.language === 'en' ? 'Difficulty' : '難易度';
  if(homeDifficultyValueEl) homeDifficultyValueEl.textContent = S.language === 'en' ? difficultyMeta.labelEn : difficultyMeta.labelJa;
  if(homeDifficultyNoteEl) homeDifficultyNoteEl.textContent = S.language === 'en'
    ? `${difficultyMeta.noteEn} Create a new world if you want to change these rules.`
    : `${difficultyMeta.noteJa} 変更したい時は新しいワールドを作成してください。`;
  if(homeDifficultySettingLabelEl) homeDifficultySettingLabelEl.textContent = S.language === 'en' ? 'Difficulty' : '難易度';
  if(homeDifficultySettingNoteEl) homeDifficultySettingNoteEl.textContent = S.language === 'en'
    ? `${getWorldConfigLabel(S.difficultyMode, S.sandboxMode, 'en')} / Create a new world to change it.`
    : `${getWorldConfigLabel(S.difficultyMode, S.sandboxMode, 'ja')} / 変更したい時は新しいワールドを作成してください。`;
  if(homeWorldCreateBtn) homeWorldCreateBtn.textContent = S.language === 'en' ? '🧭 Configure New World' : '🧭 ワールド作成';
  if(homeSettingWorldCreateBtn) homeSettingWorldCreateBtn.textContent = S.language === 'en' ? '🧭 Configure New World' : '🧭 ワールド作成';
  updateChromeLanguage();
}

window.setLanguage = function(language){
  S.language = language;
  persistLanguagePreference(language);
  updateChromeLanguage();
  updateHomeScreen(language === 'en' ? 'Language switched to English.' : '言語を日本語に切り替えました。');
  renderPanel();
  updateUI();
};

function getSafeTitleSignPosition(){
  const zones = [
    {left:[4, 22], top:[6, 22]},
    {left:[76, 92], top:[8, 24]},
    {left:[5, 24], top:[68, 86]},
    {left:[76, 93], top:[66, 84]},
    {left:[4, 16], top:[28, 60]},
    {left:[84, 94], top:[28, 60]},
  ];
  const zone = pick(zones);
  return {
    left:`${zone.left[0] + Math.random() * (zone.left[1] - zone.left[0])}%`,
    top:`${zone.top[0] + Math.random() * (zone.top[1] - zone.top[0])}%`,
  };
}

window.dropTitleSign = function(element){
  if(!element) return;
  element.classList.add('falling');
  setTimeout(() => {
    element.classList.remove('falling');
    const next = getSafeTitleSignPosition();
    element.style.top = next.top;
    element.style.left = next.left;
  }, 1400);
};

window.dropAllTitleSigns = function(){
  document.querySelectorAll('#titleFunLayer .title-sign').forEach((element, index) => {
    setTimeout(() => window.dropTitleSign(element), index * 90);
  });
};

window.exitGame = function(){
  try{ window.close(); }catch(error){}
  updateHomeScreen(S.language === 'en' ? 'You can also close the game with the window close button.' : '終了はウィンドウ右上の × でも行えます。');
};

function openHomeScreen(message=''){
  const screen = document.getElementById('homeScreen');
  if(!screen) return;
  if(S.started) autoSaveForRelog();
  if(S.draggingWork) S.draggingWork = null;
  cancelQueuedViewWork();
  hideTooltip();
  closeModal();
  disposePanelDom();
  screen.classList.add('show');
  refreshHomeAmbientArt(true);
  S.started = false;
  setSpeed(0);
  updateHomeScreen(message);
}

function closeHomeScreen(){
  const screen = document.getElementById('homeScreen');
  if(!screen) return;
  screen.classList.remove('show');
  S.started = true;
  if(S.timeMode !== 'turn' && S.speed === 0) setSpeed(1);
  lastTick = performance.now();
  if(!S.loopStarted){
    S.loopStarted = true;
    requestAnimationFrame(gameLoop);
  }
}

window.startFromHome = function(){
  runWithViewLoader(S.language === 'en' ? 'Preparing the game view...' : '内政画面を準備中...', () => {
    closeHomeScreen();
    schedulePanelRender();
    scheduleMapDraw();
    scheduleMapHudRefresh();
  });
};

window.continueFromHome = function(){
  const slot = getMostRecentSaveSlot();
  if(!getSaveMeta(slot)){
    updateHomeScreen(S.language === 'en' ? 'There is no recent save yet.' : '前回の続きはまだありません。');
    return;
  }
  runWithViewLoader(S.language === 'en' ? 'Loading your latest save...' : '前回データを読込中...', () => {
    loadGame(slot);
    closeHomeScreen();
    schedulePanelRender();
    scheduleMapDraw();
    scheduleMapHudRefresh();
  });
};

window.toggleAutoSaveFromHome = function(){
  S.autoSaveEnabled = !S.autoSaveEnabled;
  updateHomeScreen(S.autoSaveEnabled ? (S.language === 'en' ? 'Autosave enabled.' : 'オートセーブを有効化しました。') : (S.language === 'en' ? 'Autosave disabled.' : 'オートセーブを停止しました。'));
};

function resetGameState(reason=(S.language === 'en' ? 'Starting a new game.' : '新規ゲームを開始します。'), startImmediately=false, options={}){
  if(!INITIAL_SNAPSHOT?.state){
    try{ captureInitialSnapshot(); }catch(error){}
  }
  if(!INITIAL_SNAPSHOT?.state){
    openHomeScreen(S.language === 'en' ? 'Failed to prepare the initial state.' : '初期状態の準備に失敗しました。');
    return;
  }
  try{
    const preservedLanguage = getPreferredLanguage(S.language);
    const preservedSandboxMode = Object.prototype.hasOwnProperty.call(options, 'sandboxMode') ? !!options.sandboxMode : !!S.sandboxMode;
    const preservedDifficultyMode = ['easy','normal','hard'].includes(options.difficultyMode) ? options.difficultyMode : (S.difficultyMode || 'normal');
    const preservedUiMode = S.uiMode || 'desktop';
    const preservedQualityMode = S.qualityMode || 'high';
    const preservedFocusMode = !!S.focusMode;
    const preservedTimeMode = S.timeMode || 'realtime';
    const snapshot = JSON.parse(JSON.stringify(INITIAL_SNAPSHOT));
    applySaveData(snapshot);
    S.language = preservedLanguage;
    persistLanguagePreference(preservedLanguage);
    updateChromeLanguage();
    S.sandboxMode = preservedSandboxMode;
    S.difficultyMode = ['easy','normal','hard'].includes(preservedDifficultyMode) ? preservedDifficultyMode : 'normal';
    S.uiMode = preservedUiMode;
    S.qualityMode = preservedQualityMode;
    S.focusMode = preservedFocusMode;
    S.timeMode = preservedTimeMode === 'turn' ? 'turn' : 'realtime';
    S.gameOverPending = false;
    S.worldConfigLocked = true;
    S.polCapMax = S.sandboxMode ? 6000 : 400;
    if(!S.sandboxMode) S.polCap = Math.min(S.polCap, S.polCapMax);
    applyDifficultyOpeningState(S, S.difficultyMode);
    S.started = false;
    cancelQueuedViewWork();
    hideTooltip();
    closeModal();
    setSpeed(startImmediately && S.timeMode !== 'turn' ? 1 : 0);
    if(startImmediately){
      closeHomeScreen();
      lastTick = performance.now();
      schedulePanelRender();
      scheduleMapDraw();
      scheduleMapHudRefresh();
    } else {
      openHomeScreen(reason);
      updateHomeScreen(reason);
    }
  } catch(error){
    reportRuntimeError('resetGameState', error);
    openHomeScreen(S.language === 'en' ? 'Failed to start a new game.' : '最初からの開始に失敗しました。');
  }
}

window.resetFromHome = function(){
  beginWorldCreation();
};

function getSpaceMissionConfigs(){
  return {
    survey:{label:'軌道調査', cost:9000, pol:18, req:0, duration:1.2, reward:'宇宙Lv上昇 / 防衛・サイバー強化'},
    harvest:{label:'資源採掘', cost:16000, pol:28, req:1, duration:1.55, reward:'惑星ごとの新資源を採掘'},
    outpost:{label:'前哨基地建設', cost:30000, pol:44, req:2, duration:1.9, reward:'軌道施設・継続収益・宇宙株強化'},
  };
}

function getSpaceBody(bodyId){
  return spaceBodies.find(body => body.id === bodyId) || spaceBodies[0];
}

function ensureSpecialResource(materialId, originLabel){
  const material = [...uniqueMaterials, ...spaceMaterials].find(entry => entry.id === materialId);
  if(!material) return null;
  if(!resources[materialId]){
    resources[materialId] = {
      name:material.name,
      amount:0,
      max:100000,
      color:material.color,
      domestic:true,
      region:originLabel,
      price:material.price,
      priceHist:[],
      unit:'kg',
      isUnique:true,
      origin:originLabel
    };
    for(let i=0;i<120;i++) resources[materialId].priceHist.push(material.price * rand(0.9, 1.12));
  }
  return material;
}

function getRocketRequirementForMission(type, bodyId){
  const body = getSpaceBody(bodyId);
  const base = body.id === 'moon' ? 1 : body.id === 'mars' ? 2 : body.id === 'asteroid' ? 3 : 4;
  if(type === 'outpost') return base + 1;
  if(type === 'harvest') return base;
  return Math.max(1, base);
}

function getRocketDevelopmentRequirement(level){
  const orbital = getCompanyByName('オービタル重工');
  const telecom = getCompanyByName('NTテレコム');
  const defense = getCompanyByName('三星重工業');
  const materialNeeds = [
    {key:'titanium', amount:150 + level * 70},
    {key:'aluminum', amount:1200 + level * 420},
    {key:'silicon', amount:260 + level * 120},
  ];
  if(level >= 2) materialNeeds.push({key:'rareearth', amount:90 + level * 40});
  if(level >= 3) materialNeeds.push({key:'regolith_alloy', amount:80 + level * 30});
  return {
    cost:12000 + level * 12000,
    pol:18 + level * 8,
    minTech:14 + level * 8,
    minFavor:54 + level * 6,
    companies:[orbital, telecom, defense].filter(Boolean),
    materials:materialNeeds,
  };
}

function canDevelopRocket(level){
  const requirement = getRocketDevelopmentRequirement(level);
  if(S.money < requirement.cost) return {ok:false, reason:'資金不足', requirement};
  if(S.polCap < requirement.pol) return {ok:false, reason:'政治資本不足', requirement};
  if(S.techLevel < requirement.minTech) return {ok:false, reason:`国家技術力 ${requirement.minTech} 以上が必要`, requirement};
  if(requirement.companies.some(company => (company.favorability || 0) < requirement.minFavor)) return {ok:false, reason:`宇宙関連企業の好感度 ${requirement.minFavor} 以上が必要`, requirement};
  if(requirement.materials.some(entry => (resources[entry.key]?.amount || 0) < entry.amount)) return {ok:false, reason:'必要素材が不足しています', requirement};
  return {ok:true, reason:'開発できます', requirement};
}

function registerDiscoveredMaterial(materialId, origin){
  if(!S.discoveredMaterials.some(entry => entry.id === materialId && entry.region === origin)){
    S.discoveredMaterials.push({id:materialId, region:origin});
  }
}

function getSpaceMissionChance(type, bodyId){
  const cfg = getSpaceMissionConfigs()[type];
  const body = getSpaceBody(bodyId);
  if(!cfg || !body) return 0;
  const rocketReq = getRocketRequirementForMission(type, bodyId);
  const base = 0.3 - body.difficulty * 0.0038 - cfg.req * 0.05 - rocketReq * 0.05;
  const spaceCompanyBonus = ['オービタル重工','月宙システムズ','NTテレコム','三星重工業']
    .map(getCompanyByName)
    .filter(Boolean)
    .reduce((sum, company) => sum + (company.favorability || 0), 0) / 400;
  const chance = base + S.techLevel * 0.0055 + S.techBudget * 0.002 + S.cyberPower * 0.0007 + S.space.level * 0.02 + S.space.rocketLevel * 0.045 + getSpaceTechBonus() + spaceCompanyBonus - Math.max(0, S.bubbleIndex - 35) * 0.0014;
  return clamp(chance, 0.08, 0.72);
}

function resolveSpaceMission(){
  const mission = S.space.mission;
  if(!mission) return;
  const cfg = getSpaceMissionConfigs()[mission.type];
  const body = getSpaceBody(mission.bodyId);
  const chance = getSpaceMissionChance(mission.type, mission.bodyId);
  S.space.mission = null;
  if(Math.random() < chance){
    S.space.successes++;
    S.space.level = Math.max(S.space.level, mission.type === 'survey' ? 1 : mission.type === 'harvest' ? 2 : 3);
    S.space.resources += body.reward;
    S.techLevel += mission.type === 'outpost' ? 2.4 : mission.type === 'harvest' ? 1.5 : 1.2;
    S.cyberPower += mission.type === 'survey' ? 2 : 1;
    S.defPower += mission.type === 'outpost' ? 3 : 1;
    if(mission.type === 'harvest' || mission.type === 'outpost'){
      const yields = body.materials.map(id => ensureSpecialResource(id, body.name)).filter(Boolean);
      yields.forEach(material => {
        const amount = Math.round(body.reward * rand(180, 320));
        resources[material.id].amount = Math.min(resources[material.id].max * 1.8, resources[material.id].amount + amount);
        registerDiscoveredMaterial(material.id, body.id);
      });
      companies.filter(c => ['space','tech','energy','hospital'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(1.012, 1.045)));
      addEvent(`🚀 ${body.name}遠征成功。${body.materials.map(id => resources[id].name).join(' / ')} を回収`, 'good');
    } else {
      companies.filter(c => ['tech','telecom','space'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(1.01, 1.03)));
      addEvent(`🛰 ${body.name}軌道調査が成功。観測・通信基盤が強化された`, 'good');
    }
    if(mission.type === 'outpost'){
      S.space.factories++;
      S.energyBonus += 1.2;
      S.gdp *= 1.0016;
    }
  } else {
    S.space.failures++;
    S.approval = Math.max(0, S.approval - 2.4);
    S.techLevel = Math.max(0, S.techLevel - 0.4);
    companies.filter(c => c.sector === 'space' || c.sector === 'tech').forEach(c => c.price = Math.round(c.price * rand(0.968, 0.989)));
    addEvent(`💥 ${body.name}への${cfg.label}は失敗。宇宙計画が一時停滞`, 'bad');
  }
}

function renderSpace(){
  const missions = getSpaceMissionConfigs();
  const body = getSpaceBody(S.spaceFocusBody);
  const nextRocket = S.space.rocketLevel + 1;
  const rocketCheck = canDevelopRocket(nextRocket);
  const discoveredNames = [...new Set(S.discoveredMaterials.filter(entry => spaceBodies.some(bodyRef => bodyRef.id === entry.region)).map(entry => resources[entry.id]?.name).filter(Boolean))];
  let html = `<div class="section"><h3>🚀 宇宙開発局</h3>
    <div style="display:flex;gap:6px;flex-wrap:wrap;margin-bottom:8px">
      <button class="btn btn-blue ${S.currentMapMode==='space'?'active':''}" onclick="toggleMapMode('space')">宇宙マップ</button>
      <button class="btn ${S.currentMapMode==='world'?'active':''}" onclick="toggleMapMode('world')">世界マップ</button>
      <button class="btn ${S.currentMapMode==='japan'?'active':''}" onclick="toggleMapMode('japan')">日本マップ</button>
      <span class="focus-chip">現在注目: ${body.name}</span>
    </div>
    <div class="macro-grid">
      <div class="macro-card"><div class="macro-label">宇宙レベル</div><div class="macro-value">${S.space.level}</div>${buildMiniChartSvg(S.history.tech, '#64b5f6')}</div>
      <div class="macro-card"><div class="macro-label">ロケットLv</div><div class="macro-value">${S.space.rocketLevel}</div><div class="compact-note">まずロケットを開発しないと遠征できません</div></div>
      <div class="macro-card"><div class="macro-label">成功 / 失敗</div><div class="macro-value">${S.space.successes} / ${S.space.failures}</div><div class="compact-note">累計ロケット ${S.space.rockets}機</div></div>
      <div class="macro-card"><div class="macro-label">軌道施設</div><div class="macro-value">${S.space.factories}</div><div class="compact-note">週次収益 +${Math.round(S.space.resources * 0.08 + S.space.factories * 40)}億 / 中継衛星 ${S.space.orbitalRelays || 0}</div></div>
    </div>
    <div class="goal-card" style="margin-top:8px">
      <div class="goal-title"><span>🚀 ロケット開発 Lv${nextRocket}</span><span>${rocketCheck.ok ? '開発可能' : '条件不足'}</span></div>
      <div class="goal-desc">必要: 国家技術力 ${rocketCheck.requirement.minTech} / 主要企業好感度 ${rocketCheck.requirement.minFavor}+ / 素材 ${rocketCheck.requirement.materials.map(entry => `${resources[entry.key]?.name || entry.key} ${Math.round(entry.amount)}`).join(' / ')}</div>
      <div class="save-actions">
        <button class="btn btn-blue" onclick="developRocketLevel(${nextRocket})" ${rocketCheck.ok ? '' : 'disabled'}>開発 💰${formatMoneyCompact(rocketCheck.requirement.cost)} 🏛${rocketCheck.requirement.pol}</button>
        ${!rocketCheck.ok ? `<span class="compact-note">${rocketCheck.reason}</span>` : ''}
      </div>
    </div>
    <div class="headline-card" style="margin-top:8px">
      <div class="meta">${body.name}</div>
      <div class="body">${body.desc}<br>難度 ${body.difficulty} | 到達 ${body.travelWeeks}週 | 主資源 ${body.materials.map(id => (spaceMaterials.find(material => material.id === id)?.name) || id).join(' / ')}</div>
    </div>`;
  html += `<div class="goal-card" style="margin-top:8px">
    <div class="goal-title"><span>📡 地球周回通信衛星</span><span>${S.space.orbitalRelays || 0} 基</span></div>
    <div class="goal-desc">通信会社と宇宙企業が協力して地球周回衛星を打ち上げます。通信・IT・宇宙株の成長と戦時物流に効きます。</div>
    <button class="btn btn-blue" onclick="deployOrbitalRelay()" ${S.space.rocketLevel < 1 ? 'disabled' : ''}>打上げ 💰8,500億 🏛14</button>
  </div>`;
  html += `<div class="range-group">${spaceBodies.map(bodyRef => `<button class="btn btn-sm ${S.spaceFocusBody===bodyRef.id?'active':''}" onclick="selectSpaceBody('${bodyRef.id}')">${bodyRef.name}</button>`).join('')}</div>`;

  if(S.space.mission){
    const missionBody = getSpaceBody(S.space.mission.bodyId);
    const missionCfg = missions[S.space.mission.type];
    const pct = Math.round((1 - S.space.mission.weeksLeft / S.space.mission.totalWeeks) * 100);
    html += `<div class="goal-card">
      <div class="goal-title"><span>進行中: ${missionBody.name} ${missionCfg.label}</span><span>${pct}%</span></div>
      <div class="goal-desc">残り ${S.space.mission.weeksLeft}週 | 成功率 ${(getSpaceMissionChance(S.space.mission.type, S.space.mission.bodyId) * 100).toFixed(0)}%</div>
      <div class="goal-progress"><div class="fill" style="width:${pct}%"></div></div>
    </div>`;
  } else {
    Object.entries(missions).forEach(([key, mission]) => {
      const chance = getSpaceMissionChance(key, body.id);
      const locked = S.space.level < mission.req;
      const rocketReq = getRocketRequirementForMission(key, body.id);
      const totalWeeks = Math.ceil(body.travelWeeks * mission.duration + mission.req * 5);
      html += `<div class="goal-card" style="margin-top:8px">
        <div class="goal-title"><span>${mission.label}</span><span>${(chance * 100).toFixed(0)}%</span></div>
      <div class="goal-desc">必要: 宇宙Lv${mission.req} / ロケットLv${rocketReq} | ${body.name}で ${totalWeeks}週 | コスト: 💰${formatMoneyCompact(mission.cost)} / 🏛${mission.pol}<br>${mission.reward}</div>
        <button class="btn btn-blue" onclick="launchSpaceMission('${key}')" ${(locked || S.space.rocketLevel < rocketReq) ? 'disabled' : ''}>遠征開始</button>
      </div>`;
    });
  }
  html += `<div class="section" style="margin-top:8px"><h3>🪐 回収済み宇宙資源</h3>${discoveredNames.length ? discoveredNames.map(name => `<div class="trade-item"><span>${name}</span><span>活用中</span></div>`).join('') : '<div class="compact-note">まだ宇宙由来資源は回収していません。</div>'}</div>`;
  return html + '</div>';
}

window.developRocketLevel = function(level){
  const check = canDevelopRocket(level);
  if(!check.ok){ addEvent(`❌ ${check.reason}`, 'bad'); return; }
  S.money -= check.requirement.cost;
  S.polCap -= check.requirement.pol;
  check.requirement.materials.forEach(entry => {
    if(resources[entry.key]) resources[entry.key].amount = Math.max(0, resources[entry.key].amount - entry.amount);
  });
  check.requirement.companies.forEach(company => {
    company.favorability = Math.min(100, company.favorability + 3);
    company.revenue *= 1.03;
    company.price = Math.round(company.price * 1.04);
  });
  S.space.rocketLevel = Math.max(S.space.rocketLevel, level);
  S.space.level = Math.max(S.space.level, level - 1);
  addEvent(`🚀 ロケットLv${level}を開発。宇宙遠征能力が拡張された`, 'good');
  updateUI();
  renderPanel();
};

window.deployOrbitalRelay = function(){
  if(S.space.rocketLevel < 1){ addEvent('❌ まずロケット開発が必要です', 'bad'); return; }
  if(S.money < 8500 || S.polCap < 14){ addEvent('❌ 資金か政治資本が不足しています', 'bad'); return; }
  const telecom = getCompanyByName('NTテレコム');
  const spaceCo = getCompanyByName('オービタル重工');
  if((telecom?.favorability || 0) < 45 || (spaceCo?.favorability || 0) < 45){
    addEvent('❌ 通信・宇宙企業の好感度が不足しています', 'bad');
    return;
  }
  S.money -= 8500;
  S.polCap -= 14;
  S.space.orbitalRelays = (S.space.orbitalRelays || 0) + 1;
  companies.filter(company => ['telecom','tech','space'].includes(company.sector)).forEach(company => {
    company.price = Math.round(company.price * rand(1.02, 1.05));
    company.revenue *= rand(1.012, 1.03);
  });
  addEvent(`📡 地球周回通信衛星を配備。通信・宇宙・戦時物流が強化`, 'good');
  updateUI();
  renderPanel();
};

window.toggleMapMode = function(mode){
  storeCurrentView();
  if(S.draggingWork && mode !== 'japan') cancelWorkPlacement();
  if(S.draggingDisaster && mode !== S.draggingDisaster.target) cancelChaosPlacement();
  if(S.draggingStrike && mode !== 'world') cancelStrikePlacement();
  hideTooltip();
  S.currentMapMode = mode;
  if(mode === 'world' && !S.worldFocusCountry) S.worldFocusCountry = 'japan';
  loadViewForMode(mode);
  clampMapView(mode);
  scheduleMapHudRefresh();
  schedulePanelRender();
  scheduleMapDraw();
};

window.selectSpaceBody = function(bodyId){
  S.spaceFocusBody = bodyId;
  S.currentMapMode = 'space';
  loadViewForMode('space');
  scheduleMapHudRefresh();
  schedulePanelRender();
  scheduleMapDraw();
};

window.launchSpaceMission = function(type){
  const cfg = getSpaceMissionConfigs()[type];
  const body = getSpaceBody(S.spaceFocusBody);
  if(!cfg || !body) return;
  if(S.space.mission){ addEvent('⏳ すでに宇宙遠征が進行中です', 'bad'); return; }
  if(S.space.level < cfg.req){ addEvent('❌ 前提宇宙レベル不足', 'bad'); return; }
  const rocketReq = getRocketRequirementForMission(type, body.id);
  if(S.space.rocketLevel < rocketReq){ addEvent(`❌ ${body.name} へ行くにはロケットLv${rocketReq}が必要です`, 'bad'); return; }
  if(S.money < cfg.cost){ addEvent('❌ 資金不足', 'bad'); return; }
  if(S.polCap < cfg.pol){ addEvent('❌ 政治資本不足', 'bad'); return; }
  const totalWeeks = Math.ceil(body.travelWeeks * cfg.duration + cfg.req * 5);
  S.money -= cfg.cost;
  S.polCap -= cfg.pol;
  S.space.rockets++;
  S.currentMapMode = 'space';
  S.space.mission = {type, bodyId:body.id, totalWeeks, weeksLeft:totalWeeks};
  addEvent(`🚀 ${body.name}へ${cfg.label}を開始。到達まで ${totalWeeks}週`, 'good');
  updateUI();
  renderPanel();
};

function renderNews(){
  const safeNum = (value, fallback=0) => Number.isFinite(Number(value)) ? Number(value) : fallback;
  const history = S.history || {};
  let outlook = {title:'安定成長', type:'good', body:'大きな異常は確認されていません。'};
  try{
    const nextOutlook = getNationalOutlook();
    if(nextOutlook && typeof nextOutlook === 'object') outlook = nextOutlook;
  } catch(error){
    reportRuntimeError('renderNews.outlook', error);
  }
  let alerts = [];
  try{
    const nextAlerts = getStrategicAlerts();
    alerts = Array.isArray(nextAlerts) ? nextAlerts : [];
  } catch(error){
    reportRuntimeError('renderNews.alerts', error);
  }
  const headlines = Array.isArray(S.eventHistory) ? S.eventHistory.slice(0, 20) : [];
  const usdRate = safeNum(tradePartners?.[0]?.currencyRate, 0);
  const eurRate = safeNum(tradePartners?.[4]?.currencyRate, 0);
  let html = `<div class="section"><h3>📰 国家ヘッドライン</h3>
    <div class="headline-card ${outlook.type}">
      <div class="meta">国家見通し</div>
      <div class="body"><strong>${outlook.title}</strong><br>${outlook.body}</div>
    </div>`;
  alerts.slice(0, 3).forEach(alert => {
    html += `<div class="headline-card ${alert.type === 'danger' ? 'bad' : alert.type === 'good' ? 'good' : ''}">
      <div class="meta">${alert.title}</div>
      <div class="body">${alert.text}</div>
    </div>`;
  });
  html += `<div class="macro-grid" style="margin-top:8px">
    <div class="macro-card" onclick="openHistoryExplorer('gdp')" style="cursor:pointer"><div class="macro-label">GDP</div><div class="macro-value">${safeNum(S.gdp).toFixed(0)}兆</div>${buildMiniChartSvg(history.gdp, '#4ecca3')}</div>
    <div class="macro-card" onclick="openHistoryExplorer('money')" style="cursor:pointer"><div class="macro-label">国庫</div><div class="macro-value">${formatMoneyCompact(safeNum(S.money))}</div>${buildMiniChartSvg(history.money, '#4fc3f7')}</div>
    <div class="macro-card" onclick="openHistoryExplorer('approval')" style="cursor:pointer"><div class="macro-label">支持率</div><div class="macro-value">${safeNum(S.approval).toFixed(1)}%</div>${buildMiniChartSvg(history.approval, '#ffd54f')}</div>
    <div class="macro-card" onclick="openHistoryExplorer('stability')" style="cursor:pointer"><div class="macro-label">安定度</div><div class="macro-value">${safeNum(S.stability).toFixed(0)}</div>${buildMiniChartSvg(history.stability, '#ef5350')}</div>
  </div>
  <div class="macro-grid" style="margin-top:6px">
    <div class="macro-card" onclick="openHistoryExplorer('inflation')" style="cursor:pointer"><div class="macro-label">物価</div><div class="macro-value">${safeNum(S.inflation).toFixed(1)}%</div>${buildMiniChartSvg(history.inflation, '#ff8a65')}</div>
    <div class="macro-card" onclick="openHistoryExplorer('unemployment')" style="cursor:pointer"><div class="macro-label">失業率</div><div class="macro-value">${safeNum(S.unemployment).toFixed(1)}%</div>${buildMiniChartSvg(history.unemployment, '#9575cd')}</div>
    <div class="macro-card" onclick="openHistoryExplorer('debt')" style="cursor:pointer"><div class="macro-label">国債残高</div><div class="macro-value">${formatMoneyCompact(safeNum(getTotalDebt()))}</div>${buildMiniChartSvg(history.debt, '#90a4ae')}</div>
    <div class="macro-card" onclick="openHistoryExplorer('tech')" style="cursor:pointer"><div class="macro-label">技術力</div><div class="macro-value">${safeNum(S.techLevel).toFixed(1)}</div>${buildMiniChartSvg(history.tech, '#90caf9')}</div>
  </div>
  <div class="macro-grid" style="margin-top:6px">
    <div class="macro-card" onclick="openHistoryExplorer('yen')" style="cursor:pointer"><div class="macro-label">円指数</div><div class="macro-value">${safeNum(S.yenValue).toFixed(1)}</div>${buildMiniChartSvg(history.yen, '#80cbc4')}</div>
    <div class="macro-card" onclick="openHistoryExplorer('cycle')" style="cursor:pointer"><div class="macro-label">景気循環</div><div class="macro-value">${(safeNum(S.businessCycle) * 100).toFixed(1)}</div>${buildMiniChartSvg(history.cycle, '#ffb74d')}</div>
    <div class="macro-card"><div class="macro-label">通貨比較</div><div class="macro-value">USD / EUR</div><div class="compact-note">USD ¥${usdRate.toFixed(1)} / EUR ¥${eurRate.toFixed(1)}</div></div>
    <div class="macro-card"><div class="macro-label">不足資源数</div><div class="macro-value">${getShortageCount()}</div><div class="compact-note">貿易タブ上部に優先表示</div></div>
  </div>
  <div class="macro-grid" style="margin-top:6px">
    <div class="macro-card" onclick="openHistoryExplorer('unrest')" style="cursor:pointer"><div class="macro-label">社会不安</div><div class="macro-value">${safeNum(S.socialUnrest).toFixed(0)}</div>${buildMiniChartSvg(history.unrest, '#ef9a9a')}</div>
    <div class="macro-card" onclick="openHistoryExplorer('ideology')" style="cursor:pointer"><div class="macro-label">体制軸</div><div class="macro-value">${Math.round(safeNum(S.ideologyScore))}</div>${buildMiniChartSvg(history.ideology, '#a5d6a7')}</div>
  </div></div>`;
  html += '<div class="section"><h3>🗞 ニュース履歴</h3>';
  if(!headlines.length) html += '<div style="font-size:9px;color:#888">まだ記録がありません。</div>';
  headlines.forEach(item => {
    html += `<div class="headline-card ${item.type === 'bad' ? 'bad' : item.type === 'good' ? 'good' : ''}">
      <div class="meta">${item.label}</div>
      <div class="body">${item.msg}</div>
    </div>`;
  });
  html += '</div>';
  return html;
}

function renderResearch(){
  let html = `<div class="section"><h3>🧪 研究開発庁</h3><p class="compact-note">研究は対象企業を 100% 保有し、その企業の株価・売上・日本の技術力が条件を満たした時だけ開始できます。完了すると新製品・新産業・重大イベントが解放されます。なお高付加価値製品は、研究完了後も 認可 / 量産ライン / 供給契約 / 販路 の4要素が整うまで輸出できません。</p>`;
  researchCatalog.forEach(project => {
    const company = getCompanyByName(project.company);
    const status = getResearchCompanyStatus(company);
    const check = canStartResearchProject(project);
    const active = S.researchProjects.find(entry => entry.id === project.id);
    const done = hasCompletedResearch(project.id);
    html += `<div class="research-card ${!check.ok && !active && !done ? 'locked' : ''}">
      ${renderResearchHero(project, 'standard')}
      <div class="title"><span>${project.name}</span><span>${project.company}</span></div>
      <div class="meta">${project.desc}<br>必要: 100%保有 / 技術力 ${project.minTech}+ / 株価評価 ${(project.minStock * 100).toFixed(0)}% / 売上成長 ${(project.minRevenueGrowth * 100).toFixed(0)}%${project.requiresMaterial ? ` / 素材 ${resources[project.requiresMaterial]?.name || project.requiresMaterial}` : ''}</div>
      <div class="status">現在: 保有 ${hasFullControl(project.company) ? '100%' : '不足'} | 株価評価 ${(status.stockRatio * 100).toFixed(0)}% | 売上成長 ${(status.revenueRatio * 100).toFixed(0)}% | 技術力 ${S.techLevel.toFixed(1)}</div>
      <div class="actions">
        ${done ? `<span class="pill">完了</span>` : active ? `<span class="pill">進行中 残り ${active.weeksLeft}週</span>` : `<button class="btn btn-blue" onclick="startResearch('${project.id}')" ${!check.ok ? 'disabled' : ''}>開始 💰${formatMoneyCompact(project.cost)} 🏛${project.pol}</button>`}
        ${project.unlock ? `<span class="pill">解放: ${productCatalog[project.unlock]?.name || project.unlock}</span>` : ''}
        ${!done && !active && !check.ok ? `<span class="compact-note">${check.reason}</span>` : ''}
      </div>
    </div>`;
  });
  html += `</div><div class="section"><h3>☣ 特殊病原体研究</h3><p class="compact-note">武野薬品工業を完全支配し、十分な技術力と企業評価を満たすと人工病原体を開発できます。完成後に任意で散布可能です。</p>`;
  pathogenCatalog.forEach(pathogen => {
    const company = getCompanyByName(pathogen.company);
    const status = getResearchCompanyStatus(company);
    const active = S.researchProjects.find(entry => entry.id === pathogen.id);
    const check = canStartResearchProject(pathogen);
    const unlocked = S.bioWeaponUnlocks.includes(pathogen.id);
    html += `<div class="research-card ${!check.ok && !active && !unlocked ? 'locked' : ''}">
      ${renderResearchHero(pathogen, 'pathogen')}
      <div class="title"><span>${pathogen.name}</span><span>協力: ${pathogen.company}</span></div>
      <div class="meta">${pathogen.desc}<br>必要: 100%保有 / 技術力 ${pathogen.minTech}+ / 株価評価 ${(pathogen.minStock * 100).toFixed(0)}% / 売上成長 ${(pathogen.minRevenueGrowth * 100).toFixed(0)}%${pathogen.requiresMaterial ? ` / 素材 ${resources[pathogen.requiresMaterial]?.name || pathogen.requiresMaterial}` : ''}</div>
      <div class="status">現在: 保有 ${hasFullControl(pathogen.company) ? '100%' : '不足'} | 株価評価 ${(status.stockRatio * 100).toFixed(0)}% | 売上成長 ${(status.revenueRatio * 100).toFixed(0)}% | 技術力 ${S.techLevel.toFixed(1)}</div>
      <div class="actions">
        ${active ? `<span class="pill">進行中 残り ${active.weeksLeft}週</span>` : unlocked ? `<button class="btn" style="border-color:#e94560" onclick="deployPathogen('${pathogen.id}')">散布</button>` : `<button class="btn" style="border-color:#e94560" onclick="startResearch('${pathogen.id}','pathogen')" ${!check.ok ? 'disabled' : ''}>研究 💰${formatMoneyCompact(pathogen.cost)} 🏛${pathogen.pol}</button>`}
        ${!active && !unlocked && !check.ok ? `<span class="compact-note">${check.reason}</span>` : ''}
      </div>
    </div>`;
  });
  if(S.unlockedProducts.length){
    html += `<div class="section"><h3>📦 研究で解放済みの輸出品</h3>`;
    html += S.unlockedProducts.map(productId => {
      const product = productCatalog[productId];
      const status = getProductCommercializationStatus(productId);
      if(!product) return '';
      if(!product.requires?.length){
        return `<div class="trade-item"><span>${product.name}</span><span>標準製品</span></div>`;
      }
      const detail = status?.detail || {certification:0,line:0,supply:0,channel:0};
      return `<div style="padding:8px 0;border-bottom:1px solid rgba(255,255,255,.05)">
        <div class="trade-item" style="border-bottom:none;padding:0">
          <span>${product.name}</span>
          <span>${getProductIndustrializationLabel(productId)}</span>
        </div>
        <div class="compact-note" style="margin-top:4px">認可 ${Math.round(detail.certification * 100)}% / 量産ライン ${Math.round(detail.line * 100)}% / 供給契約 ${Math.round(detail.supply * 100)}% / 販路 ${Math.round(detail.channel * 100)}%</div>
        ${status?.missing?.length ? `<div class="compact-note" style="margin-top:4px;color:#ffcf91">不足: ${status.missing.join(' / ')}</div>` : `<div class="compact-note" style="margin-top:4px;color:#8ff0bf">必要条件は概ね整っています。量産化の最終立ち上げ中です。</div>`}
      </div>`;
    }).join('');
    html += `</div>`;
  }
  return html;
}

window.startResearch = function(id, kind='standard'){
  const project = (kind === 'pathogen' ? pathogenCatalog : researchCatalog).find(entry => entry.id === id);
  if(!project) return;
  if(S.researchProjects.some(entry => entry.id === id)){ addEvent('⏳ その研究はすでに進行中です', 'bad'); return; }
  const check = canStartResearchProject(project);
  if(!check.ok){ addEvent(`❌ 研究開始不可: ${check.reason}`, 'bad'); return; }
  S.money -= project.cost;
  S.polCap -= project.pol;
  S.researchProjects.push({...project, kind, weeksLeft:project.weeks, totalWeeks:project.weeks});
  addEvent(`🧪 研究「${project.name}」を開始`, kind === 'pathogen' ? 'bad' : 'good');
  renderPanel();
  updateUI();
};

window.deployPathogen = function(id){
  const pathogen = pathogenCatalog.find(entry => entry.id === id);
  if(!pathogen || !S.bioWeaponUnlocks.includes(id)){ addEvent('❌ まだ病原体研究が完了していません', 'bad'); return; }
  triggerBioOutbreak(pathogen);
  renderPanel();
  updateUI();
};

function getHistoryMeta(metric){
  const map = {
    gdp:{label:'GDP', suffix:'兆', color:'#4ecca3'},
    money:{label:'国庫', suffix:'円', color:'#4fc3f7'},
    polCap:{label:'政治資本', suffix:'', color:'#ce93d8'},
    approval:{label:'支持率', suffix:'%', color:'#ffd54f'},
    pop:{label:'人口', suffix:'万', color:'#81c784'},
    stability:{label:'安定度', suffix:'', color:'#ef5350'},
    inflation:{label:'物価', suffix:'%', color:'#ff8a65'},
    unemployment:{label:'失業率', suffix:'%', color:'#9575cd'},
    debt:{label:'国債残高', suffix:'円', color:'#90a4ae'},
    bubble:{label:'市場熱', suffix:'%', color:'#e94560'},
    tech:{label:'技術力', suffix:'', color:'#90caf9'},
    yen:{label:'円指数', suffix:'', color:'#80cbc4'},
    def:{label:'防衛力', suffix:'', color:'#64b5f6'},
    cyber:{label:'サイバー', suffix:'', color:'#7e57c2'},
    cycle:{label:'景気循環', suffix:'', color:'#ffb74d'},
    unrest:{label:'社会不安', suffix:'', color:'#ef9a9a'},
    ideology:{label:'体制軸', suffix:'', color:'#a5d6a7'},
    nuclear:{label:'核戦力', suffix:'', color:'#ef9a9a'},
  };
  return map[metric] || map.gdp;
}

function formatHistoryValue(metric, value){
  const meta = getHistoryMeta(metric);
  if(metric === 'money' || metric === 'debt') return formatMoneyCompact(value);
  if(metric === 'gdp') return `${Number(value).toFixed(1)}兆`;
  if(metric === 'approval' || metric === 'bubble' || metric === 'inflation' || metric === 'unemployment') return `${Number(value).toFixed(1)}${meta.suffix}`;
  if(metric === 'pop') return `${Number(value).toFixed(1)}万`;
  if(metric === 'polCap' || metric === 'def' || metric === 'cyber' || metric === 'nuclear') return `${Number(value).toFixed(metric === 'cyber' ? 1 : 0)}`;
  if(metric === 'cycle') return `${(Number(value) * 100).toFixed(1)}`;
  if(metric === 'ideology'){
    const summary = getIdeologySummary(Number(value));
    return `${Math.round(Number(value))} / ${summary.title}`;
  }
  return `${Number(value).toFixed(meta.suffix ? 1 : 1)}${meta.suffix}`;
}

function getSvgChartMarkerPosition(wrap, index, length, value, min, range){
  const width = Math.max(1, wrap?.clientWidth || wrap?.getBoundingClientRect?.().width || 1);
  const height = Math.max(1, wrap?.clientHeight || wrap?.getBoundingClientRect?.().height || 1);
  const safeLength = Math.max(1, length);
  const x = safeLength <= 1 ? width / 2 : (index / Math.max(1, safeLength - 1)) * width;
  const yPct = 100 - ((value - min) / range * 84 + 8);
  const y = clamp((yPct / 100) * height, 0, height);
  return {x, y};
}

function bindHistoryExplorer(metric, series, labels, meta){
  const wrap = document.getElementById('historyChartWrap');
  const readout = document.getElementById('historyHoverReadout');
  const line = document.getElementById('historyCursorLine');
  const marker = document.getElementById('historyPointMarker');
  if(!wrap || !readout || !line || !marker || !series.length) return;
  const bounds = getChartSeriesBounds(series, meta);
  const min = bounds.min;
  const max = bounds.max;
  const range = max - min || 1;
  const setIndex = (index, visible=true) => {
    const idx = clamp(index, 0, series.length - 1);
    const pos = getSvgChartMarkerPosition(wrap, idx, series.length, series[idx], min, range);
    readout.innerHTML = `<span>${labels[idx] || '-'}</span><span style="color:${meta.color};font-weight:700">${formatHistoryValue(metric, series[idx])}</span>`;
    line.style.left = `${pos.x}px`;
    line.style.opacity = visible ? '1' : '0';
    marker.style.left = `${pos.x}px`;
    marker.style.top = `${pos.y}px`;
    marker.style.borderColor = meta.color;
    marker.style.opacity = visible ? '1' : '0';
  };
  setIndex(series.length - 1, true);
  wrap.addEventListener('mousemove', event => {
    const rect = wrap.getBoundingClientRect();
    const pct = clamp((event.clientX - rect.left) / Math.max(1, rect.width), 0, 1);
    const index = Math.round(pct * Math.max(0, series.length - 1));
    setIndex(index, true);
  });
  wrap.addEventListener('mouseleave', () => setIndex(series.length - 1, true));
}

window.openHistoryExplorer = function(metric){
  S.historyMetric = metric;
  const meta = getHistoryMeta(metric);
  const modal = document.getElementById('actionModal');
  document.getElementById('modalTitle').textContent = `📈 ${meta.label} 履歴`;
  const series = S.history[metric] || [];
  const labels = S.history.weeks || [];
  const usedSeries = S.historyRange >= 9999 ? series : series.slice(-S.historyRange);
  const usedLabels = S.historyRange >= 9999 ? labels : labels.slice(-S.historyRange);
  const chartOptions = metric === 'ideology'
    ? {full:true, midLineValue:0, lineWidth:0.95, padRatio:0.5, padMagnitude:0.14, minPadding:12}
    : {full:true, lineWidth:0.95, padRatio:0.5, padMagnitude:0.12, minPadding:metric === 'cycle' ? 0.12 : 1};
  const svg = buildMiniChartSvg(usedSeries, meta.color, chartOptions).replace('class="mini-chart"', 'class="history-chart-svg"');
  document.getElementById('modalContent').innerHTML = `<div class="range-group">
      ${[[52,'1年'],[156,'3年'],[260,'5年'],[520,'10年'],[9999,'全期間']].map(([range,label]) => `<button class="btn btn-sm ${S.historyRange===range?'active':''}" onclick="setHistoryRange('${metric}',${range})">${label}</button>`).join('')}
    </div>
    <div class="history-hover-readout" id="historyHoverReadout"><span>${usedLabels[usedLabels.length-1] || '-'}</span><span style="color:${meta.color};font-weight:700">${formatHistoryValue(metric, usedSeries[usedSeries.length-1] || 0)}</span></div>
    <div class="history-chart-wrap" id="historyChartWrap">
      <div class="history-cursor-line" id="historyCursorLine"></div>
      <div class="history-point-marker" id="historyPointMarker"></div>
      ${svg}
    </div>
  ${metric === 'ideology' ? `<div class="history-axis-note"><span><strong>上</strong> 民主主義寄り</span><span><strong>0</strong> 中央均衡</span><span><strong>下</strong> 社会主義・統制寄り</span></div>` : ''}
    <div class="history-legend"><span>最古: ${usedLabels[0] || '-'}</span><span>最新: ${usedLabels[usedLabels.length-1] || '-'}</span><span>サンプル数: ${usedSeries.length}</span><span>表示範囲: ${S.historyRange >= 9999 ? '全期間' : `${Math.round(S.historyRange / 52)}年`}</span></div>`;
  meta.fixedMin = chartOptions.fixedMin;
  meta.fixedMax = chartOptions.fixedMax;
  bindHistoryExplorer(metric, usedSeries, usedLabels, meta);
  modal.classList.add('show');
};

window.setHistoryRange = function(metric, range){
  S.historyRange = range;
  openHistoryExplorer(metric);
};

function renderWorld(){
  const focused = getWorldCountry(S.worldFocusCountry);
  const snapshot = getWorldCountrySnapshot(focused);
  const partner = getWorldCountryPartner(focused);
  const spyStatus = getSpyIntelStatus(focused.id);
  const activeWarCountry = getWarStateCountry();
  const focusedIntel = canViewWarIntel(focused.id) ? getCountryMilitaryIntel(focused.id) : null;
  const orderedCountries = [focused, ...worldCountries.filter(country => country.id !== focused.id)];
  let html = `<div class="section"><h3>🌐 世界マップ</h3>
    <div style="display:flex;gap:6px;flex-wrap:wrap;margin-bottom:8px">
      <button class="btn ${S.currentMapMode==='japan'?'active':''}" onclick="toggleMapMode('japan')">日本マップ</button>
      <button class="btn btn-blue ${S.currentMapMode==='world'?'active':''}" onclick="toggleMapMode('world')">世界マップ</button>
      <button class="btn ${S.currentMapMode==='space'?'active':''}" onclick="toggleMapMode('space')">宇宙マップ</button>
      <span class="focus-chip">現在注目: ${focused.name}</span>
    </div>
    <div class="map-note-card">
      <div class="title">${focused.name}</div>
      <div class="body">${focused.note}<br>地図上のノードを直接押すと、その国へフォーカスして詳細や外交操作へすぐ入れます。</div>
    </div>
    <div class="macro-grid">
      <div class="macro-card"><div class="macro-label">GDP</div><div class="macro-value">${snapshot.gdp ? `${Math.round(snapshot.gdp)}兆` : 'N/A'}</div><div class="compact-note">${focused.id === 'japan' ? '日本本体の総生産' : '相手国の経済規模指標'}</div></div>
      <div class="macro-card"><div class="macro-label">友好 / 姿勢</div><div class="macro-value">${snapshot.relation}</div><div class="compact-note">${snapshot.label}</div></div>
      <div class="macro-card"><div class="macro-label">サイバー</div><div class="macro-value">${Number(snapshot.cyber).toFixed(0)}</div><div class="compact-note">情報戦・交易条件への影響</div></div>
      <div class="macro-card"><div class="macro-label">通貨 / 核戦力</div><div class="macro-value">${snapshot.currencyCode || 'JPY'} / ${snapshot.nuke}</div><div class="compact-note">1${snapshot.currencyCode || 'JPY'} ≒ ¥${Number(snapshot.currencyRate || 1).toFixed(2)}${snapshot.nuclearScars ? `<br>戦災後遺症 ${Number(snapshot.nuclearScars).toFixed(1)}` : ''}</div></div>
      <div class="macro-card"><div class="macro-label">防御戦力</div><div class="macro-value">${focusedIntel ? Math.round(focusedIntel.power) : '不明'}</div><div class="compact-note">${focusedIntel ? `歩兵 ${focusedIntel.units.infantry} / 戦車 ${focusedIntel.units.tanks}` : '友好度が高いか交戦中で開示'}</div></div>
      <div class="macro-card"><div class="macro-label">諜報網</div><div class="macro-value">${spyStatus ? spyStatus.weeksLeft : 0}週</div><div class="compact-note">${focused.id === 'japan' ? `保有諜報班 ${S.militaryInventory.spies || 0}` : `潜入中 ${spyStatus?.activeMissions || 0} / 警戒度 ${spyStatus ? spyStatus.alert.toFixed(1) : '0.0'}`}</div></div>
    </div>
    <div class="world-sector-list">${focused.displaySectors.map(sectorId => {
      const sector = sectors[sectorId];
      return sector ? `<span class="world-sector-pill">${sector.icon}${sector.name}</span>` : '';
    }).join('')}</div>
    <div class="world-link-row">
      <button class="btn btn-sm btn-blue" onclick="focusWorldCountry('${focused.id}', {switchMap:true, openModal:true})">詳細を開く</button>
      ${partner ? `<button class="btn btn-sm btn-green" onclick="improveDiplomacy(${focused.partnerIdx}); renderPanel();">外交改善</button>` : ''}
      ${partner ? `<button class="btn btn-sm" onclick="worsenDiplomacy(${focused.partnerIdx}); renderPanel();">敵対工作</button>` : ''}
      ${partner ? `<button class="btn btn-sm btn-yellow" onclick="declareWar('${focused.id}')">宣戦布告</button>` : ''}
      ${partner ? `<button class="btn btn-sm btn-purple" onclick="sendSpyMission('${focused.id}',1); renderPanel();">諜報班x1</button>` : ''}
      ${partner ? `<button class="btn btn-sm" onclick="S.currentPanel='trade'; renderPanel();">貿易タブへ</button>` : ''}
    </div>`;
  if(S.activeWar && S.activeWar.countryId === focused.id){
    const war = S.activeWar;
    const jpFrontPower = Math.round(getUnitsPower(war.jpFront, 'jp'));
    const enemyFrontPower = Math.round(getUnitsPower(war.enemyFront, 'enemy', partner));
    html += `<div class="section" style="margin-top:8px"><h3>⚔ 交戦中</h3>
      <div class="macro-grid">
        <div class="macro-card"><div class="macro-label">輸送距離</div><div class="macro-value">${war.travelWeeks}週</div><div class="compact-note">海運・通信・軌道中継で短縮</div></div>
        <div class="macro-card"><div class="macro-label">戦況</div><div class="macro-value">${war.warScore.toFixed(1)}</div><div class="compact-note">正なら優勢、負なら劣勢</div></div>
        <div class="macro-card"><div class="macro-label">日本前線</div><div class="macro-value">${jpFrontPower}</div><div class="compact-note">歩 ${war.jpFront.infantry} / 戦 ${war.jpFront.tanks} / 航 ${war.jpFront.fighters}</div></div>
        <div class="macro-card"><div class="macro-label">敵前線</div><div class="macro-value">${enemyFrontPower}</div><div class="compact-note">${canViewWarIntel(focused.id) ? `歩 ${war.enemyFront.infantry} / 戦 ${war.enemyFront.tanks} / 航 ${war.enemyFront.fighters}` : '不明'}</div></div>
      </div>
      ${renderWarDispatchControls(focused.id, 'panel')}
      <div class="save-actions" style="margin-top:8px">
        <button class="btn btn-yellow" onclick="openWarBattleCommand()">会戦画面</button>
        <button class="btn btn-yellow" onclick="fightWarBattle(); renderPanel();">たたかう</button>
        <button class="btn" onclick="returnWarUnits()">戻る</button>
      </div>
      <div class="compact-note" style="margin-top:8px">日本予備: 歩 ${S.militaryInventory.infantry} / 戦 ${S.militaryInventory.tanks} / 航 ${S.militaryInventory.fighters} / 艦 ${S.militaryInventory.ships} / 無 ${S.militaryInventory.drones}<br>敵予備: ${canViewWarIntel(focused.id) ? `歩 ${war.enemyReserves.infantry} / 戦 ${war.enemyReserves.tanks} / 航 ${war.enemyReserves.fighters}` : '不明'} / 輸送中 日本 ${war.jpDispatches.length}便・敵 ${war.enemyDispatches.length}便</div>
      ${(war.logs || []).map(entry => `<div class="advisor-item ${entry.type === 'bad' ? 'danger' : entry.type === 'good' ? '' : 'warn'}">${entry.week} ${entry.text}</div>`).join('') || ''}
    </div>`;
  }
  if(partner){
    html += `<div class="compact-note" style="margin-top:8px">実質GDP ${snapshot.actualGdp.toFixed(2)}兆ドル / 成長率 ${Number(snapshot.growth || 0).toFixed(1)}% / 物価 ${Number(snapshot.inflation || 0).toFixed(1)}% / 失業率 ${Number(snapshot.unemployment || 0).toFixed(1)}%<br>輸出候補: ${partner.sells.map(item => resources[item]?.name || productCatalog[item]?.name || item).join(' / ')}<br>需要候補: ${partner.buys.map(item => resources[item]?.name || productCatalog[item]?.name || item).join(' / ')}</div>`;
  }
  if(activeWarCountry && (!S.activeWar || S.activeWar.countryId !== focused.id)){
    html += `<div class="goal-card" style="margin-top:8px">
      <div class="goal-title"><span>⚔ 交戦中</span><span>${activeWarCountry.name}</span></div>
      <div class="goal-desc">現在の主戦場は ${activeWarCountry.name}。その国を注目すると送軍・会戦・撤退が操作できます。</div>
    </div>`;
  }
  html += `</div><div class="section"><h3>🌍 国家一覧</h3>`;
  orderedCountries.forEach(country => {
    const countrySnapshot = getWorldCountrySnapshot(country);
    const countryPartner = getWorldCountryPartner(country);
    const activeDeals = typeof country.partnerIdx === 'number' ? S.tradeDeals.filter(deal => deal.partnerIdx === country.partnerIdx).length : 0;
    const unlocked = canInvestInCountry(country.id);
    html += `<div class="world-country-card" style="border-color:${country.id === 'japan' ? 'rgba(78,204,163,.3)' : countrySnapshot.relation >= 72 ? 'rgba(97,183,255,.24)' : countrySnapshot.relation >= 48 ? 'rgba(255,193,86,.22)' : 'rgba(233,92,96,.24)'}">
      <div class="top">
        <div class="name">${country.name}</div>
        <div class="pill">${country.id === 'japan' ? '本拠地' : unlocked ? '投資解禁' : `友好 ${countrySnapshot.relation}`}</div>
      </div>
      <div class="meta">${country.note}</div>
      <div class="bars">
        <div class="barbox"><div class="barlabel">GDP</div><div class="barvalue">${Math.round(countrySnapshot.gdp || 0)}兆</div></div>
        <div class="barbox"><div class="barlabel">Cyber</div><div class="barvalue">${Number(countrySnapshot.cyber).toFixed(0)}</div></div>
        <div class="barbox"><div class="barlabel">Routes</div><div class="barvalue">${activeDeals}</div></div>
      </div>
      <div class="world-sector-list">${country.displaySectors.map(sectorId => {
        const sector = sectors[sectorId];
        return sector ? `<span class="world-sector-pill">${sector.icon}${sector.name}</span>` : '';
      }).join('')}</div>
      <div class="world-link-row">
        <button class="btn btn-sm btn-blue" onclick="focusWorldCountry('${country.id}', {switchMap:true, switchPanel:true})">地図で注目</button>
        <button class="btn btn-sm" onclick="openWorldCountryModal('${country.id}')">詳細</button>
        ${countryPartner ? `<button class="btn btn-sm btn-green" onclick="improveDiplomacy(${country.partnerIdx}); renderPanel();">外交改善</button>` : ''}
        ${countryPartner ? `<button class="btn btn-sm" onclick="worsenDiplomacy(${country.partnerIdx}); renderPanel();">敵対工作</button>` : ''}
        ${countryPartner ? `<button class="btn btn-sm btn-yellow" onclick="declareWar('${country.id}')">宣戦</button>` : ''}
      </div>
      ${unlocked ? `<div class="status-badge good" style="margin-top:8px">戦勝済み: 海外株タブで投資可能</div>` : ''}
    </div>`;
  });
  html += '</div>';
  return html;
}

const baseRenderNews = renderNews;
renderNews = function(){
  try{
    return baseRenderNews();
  } catch(error){
    reportRuntimeError('renderNews', error);
    return `<div class="section"><h3>📰 ニュース</h3><div class="compact-note">ニュースの描画中にエラーが起きました。いったん他のタブへ移動してから戻ると復帰する場合があります。</div></div>`;
  }
};

function renderForeignMarket(){
  ensureForeignCompanies();
  const unlockedCountries = S.warVictories.map(countryId => getWorldCountry(countryId)).filter(Boolean);
  const foreignValue = Object.entries(S.foreignOwnedStocks).reduce((sum, [companyId, qty]) => {
    const company = S.foreignCompanies.find(entry => entry.id === companyId);
    return sum + (company ? company.price * qty : 0);
  }, 0);
  let html = `<div class="section"><h3>🌍 戦勝国の海外株市場</h3>
    <div class="headline-card">
      <div class="meta">海外ポートフォリオ</div>
      <div class="body">評価額 ${formatMoneyCompact(foreignValue)} / 解禁国 ${unlockedCountries.length} / 年次配当は毎年末にまとめて入金されます。</div>
    </div>`;
  if(!unlockedCountries.length){
    html += `<div class="compact-note">まだ戦勝国がありません。世界タブで友好度の低い国に対して敵対工作と宣戦布告を行い、勝利すると当地企業へ投資できます。</div></div>`;
    return html;
  }
  unlockedCountries.forEach(country => {
    const companiesForCountry = S.foreignCompanies.filter(company => company.countryId === country.id);
    html += `<div class="section"><h3>${country.name}</h3>
      ${companiesForCountry.map(company => {
        const owned = S.foreignOwnedStocks[company.id] || 0;
        const ratio = clamp(owned / Math.max(1, company.sharesOutstanding) * 100, 0, 100);
        return `<div class="foreign-stock-card">
          <div class="work-row">
            <div>
              <div style="font-size:14px;font-weight:700">${sectors[company.sector]?.icon || '🌍'} ${company.name}</div>
              <div class="work-meta">株価 ¥${Math.round(company.price).toLocaleString()} / 売上 ¥${Math.round(company.revenue).toLocaleString()} / 保有 ${ratio.toFixed(2)}%</div>
            </div>
            <div class="save-actions">
              <button class="btn btn-sm" onclick="openForeignCompanyDetail('${company.id}')">詳細</button>
              <button class="btn btn-sm btn-green" onclick="buyForeignStock('${company.id}', 10)">10株</button>
              <button class="btn btn-sm btn-green" onclick="buyForeignStock('${company.id}', 50)">50株</button>
              <button class="btn btn-sm btn-green" onclick="buyForeignStock('${company.id}', 100)">100株</button>
              <button class="btn btn-sm btn-green" onclick="buyForeignStock('${company.id}', 250)">250株</button>
            </div>
          </div>
        </div>`;
      }).join('')}
    </div>`;
  });
  return html;
}

function renderCompare(){
  const metrics = [
    {label:'GDP', japan:S.gdp, field:'gdp', suffix:'兆'},
    {label:'技術力', japan:S.techLevel, field:'techLevel', suffix:''},
    {label:'サイバー', japan:S.cyberPower, field:'cyberPower', suffix:''},
    {label:'核戦力', japan:S.nukePower, field:'nukePower', suffix:''},
    {label:'安定度', japan:S.stability, field:'stability', suffix:''},
  ];
  let html = `<div class="section"><h3>🌍 日本と他国の比較</h3><p class="compact-note">経済・技術・軍事・安定度を並べて見られます。</p>
    <div class="world-link-row" style="margin-bottom:8px">
      <button class="btn btn-sm btn-blue" onclick="toggleMapMode('world'); S.currentPanel='world'; renderPanel();">世界マップへ</button>
      <button class="btn btn-sm" onclick="toggleMapMode('japan')">日本マップへ戻る</button>
    </div>`;
  tradePartners.forEach(partner => {
    html += `<div class="goal-card">
      <div class="goal-title"><span>${partner.name}</span><span>友好 ${partner.relation}</span></div>
      ${metrics.map(metric => {
        const other = partner[metric.field] || partner.economy;
        const maxVal = Math.max(metric.japan, other, 1);
        const jpPct = metric.japan / maxVal * 100;
        const otPct = other / maxVal * 100;
        return `<div style="margin:6px 0">
          <div class="trade-item"><span>${metric.label}</span><span>日 ${metric.japan.toFixed ? metric.japan.toFixed( metric.field === 'gdp' ? 0 : 1) : metric.japan}${metric.suffix} / 相手 ${other.toFixed ? other.toFixed(metric.field === 'gdp' ? 0 : 1) : other}${metric.suffix}</span></div>
          <div style="display:flex;gap:6px">
            <div style="height:8px;flex:${jpPct};background:#4ecca3;border-radius:999px"></div>
            <div style="height:8px;flex:${otPct};background:#e94560;border-radius:999px"></div>
          </div>
        </div>`;
      }).join('')}
    </div>`;
  });
  return html + '</div>';
}

function getIdeologySummary(score=S.ideologyScore){
  if(score >= 55) return {title:'自由民主主義', body:'市場と民間成長を重視。金融・IT・小売が伸びやすく、西側諸国と親和的。', color:'#8bd0ff'};
  if(score >= 20) return {title:'穏健民主', body:'民間主導だが公共投資も重視する中道。幅広い産業が安定しやすい。', color:'#b7f5d0'};
  if(score > -20) return {title:'混合経済', body:'国家と市場の比率が拮抗。外交面も比較的バランス型。', color:'#ffe49b'};
  if(score > -55) return {title:'計画経済寄り', body:'公共・重工・インフラを優先。国営色が強まり、国家統制産業が伸びやすい。', color:'#ffbd9a'};
  return {title:'国家統制体制', body:'国家計画と統制を最重視。重工・防衛・エネルギーが強いが自由経済圏との相性は悪い。', color:'#ff9fa6'};
}

function getLawCooldownWeeks(){
  return Math.max(52, Math.round((S.lawChangeIntervalYears || 3) * 52));
}

function getYearWeekFromTotalWeeks(totalWeeks){
  const absoluteWeeks = Math.max(0, Math.round(totalWeeks || 0));
  return {
    year:2025 + Math.floor(absoluteWeeks / 52),
    week:absoluteWeeks % 52 + 1,
  };
}

function isLawChangeLocked(){
  return (S.totalWeeks || 0) < (S.nextLawChangeWeek || 0);
}

function getLawCooldownLabel(){
  if(!isLawChangeLocked()) return S.language === 'en' ? 'Available now' : '今すぐ変更可能';
  const unlock = getYearWeekFromTotalWeeks(S.nextLawChangeWeek || 0);
  return S.language === 'en'
    ? `Next change: ${formatWeekText(unlock.year, unlock.week)}`
    : `次回変更可能: ${formatWeekText(unlock.year, unlock.week)}`;
}

function getCountryIdeologyPreference(partnerId){
  const map = {
    usa:0.85,
    germany:0.72,
    australia:0.66,
    korea:0.38,
    india:0.16,
    brazil:0.1,
    chile:0.14,
    indonesia:0.04,
    saudi:-0.08,
    china:-0.78,
  };
  return map[partnerId] ?? 0;
}

function getCompanyIdeologyModifier(company){
  const axis = clamp(S.ideologyScore / 100, -1, 1);
  const sectorWeights = {
    finance:1.25,
    tech:1.05,
    retail:0.72,
    telecom:0.52,
    auto:0.22,
    pharma:0.12,
    hospital:-0.24,
    food:-0.18,
    energy:-0.58,
    construction:-0.66,
    defense:-0.82,
    shipping:-0.52,
    mining:-0.36,
    space:0.12,
  };
  return axis * (sectorWeights[company.sector] || 0) * 0.036;
}

function getLawCatalog(){
  return [
    {id:'market_reform', name:'市場自由化', shift:16, cost:7000, pol:26, desc:'規制緩和で民間投資を促進。金融・IT・小売に追い風。'},
    {id:'civil_rights', name:'市民権拡充', shift:10, cost:4200, pol:18, desc:'自由度と開放性を強め、支持率と西側友好を押し上げる。'},
    {id:'mixed_balance', name:'均衡調整', shift:0, setMode:'toward_zero', cost:5200, pol:16, desc:'極端な体制傾向を和らげ、経済と外交のバランスを整える。'},
    {id:'industrial_plan', name:'産業計画法', shift:-14, cost:7600, pol:26, desc:'国家主導で重工・インフラ・エネルギーへ資源配分を寄せる。'},
    {id:'state_control', name:'国有化推進', shift:-24, cost:9800, pol:34, desc:'公共・重工・防衛へ強い追い風。ただし自由経済圏との関係は悪化しやすい。'},
  ];
}

window.enactLaw = function(lawId){
  const law = getLawCatalog().find(entry => entry.id === lawId);
  if(!law) return;
  if(isLawChangeLocked()){
    addEvent(`❌ 法律は ${S.lawChangeIntervalYears}年ごとにしか変更できません。${getLawCooldownLabel()}`, 'bad');
    return;
  }
  if(S.money < law.cost){ addEvent('❌ 資金不足', 'bad'); return; }
  if(S.polCap < law.pol){ addEvent('❌ 政治資本不足', 'bad'); return; }
  S.money -= law.cost;
  S.polCap -= law.pol;
  const previousScore = S.ideologyScore;
  let nextScore = S.ideologyScore;
  if(law.setMode === 'toward_zero'){
    nextScore = Math.round(lerp(S.ideologyScore, 0, 0.55));
  } else {
    nextScore = clamp(S.ideologyScore + law.shift, -100, 100);
  }
  const delta = nextScore - previousScore;
  S.ideologyScore = nextScore;
  S.nextLawChangeWeek = (S.totalWeeks || 0) + getLawCooldownWeeks();
  S.lastLawChangeId = law.id;
  S.ideologyDrift = clamp(delta * 0.28, -18, 18);
  const radicalShift = Math.abs(delta);
  S.approval = clamp(S.approval + (radicalShift > 14 ? -2.4 : 0.9) - radicalShift * 0.03, 0, 100);
  S.stability = clamp(S.stability + (radicalShift > 12 ? -2.8 : 1.2) - radicalShift * 0.02, 0, 100);
  S.socialUnrest = clamp(S.socialUnrest + Math.max(0, radicalShift - 6) * 0.46, 0, 100);
  S.businessCycle = clamp(S.businessCycle + delta * 0.0018, -0.4, 0.4);
  S.yenValue = clamp(S.yenValue + delta * 0.08, 72, 160);
  tradePartners.forEach(partner => {
    const pref = getCountryIdeologyPreference(partner.id);
    const currentGap = Math.abs(previousScore / 100 - pref);
    const nextGap = Math.abs(S.ideologyScore / 100 - pref);
    const relationShift = (currentGap - nextGap) * 26;
    partner.relation = clamp(partner.relation + relationShift, 0, 100);
    partner.hostility = clamp((partner.hostility || 0) - relationShift * 0.9, 0, 100);
    partner.stability = clamp(partner.stability + relationShift * 0.12, 0, 100);
  });
  companies.forEach(company => {
    const lawShock = getCompanyIdeologyModifier(company);
    company.revenue = Math.max(120, Math.round(company.revenue * clamp(1 + lawShock * 7.5, 0.82, 1.22)));
    const nextPrice = Math.round(company.price * clamp(1 + lawShock * 11 + rand(-0.012, 0.012), 0.78, 1.28));
    company.price = clamp(nextPrice, getCompanyDynamicFloor(company), getCompanySoftCeiling(company) * 1.18);
    company.favorability = clamp((company.favorability || 50) + lawShock * 120, 0, 100);
  });
  addEvent(`⚖ 法律「${law.name}」を制定。体制軸が ${Math.round(S.ideologyScore)} に変化。${getLawCooldownLabel()}`, 'good');
  renderPanel();
  updateUI();
};

function renderLaw(){
  const summary = getIdeologySummary();
  const locked = isLawChangeLocked();
  const chart = buildMiniChartSvg(S.history.ideology || [], '#a5d6a7', {full:true, fixedMin:-100, fixedMax:100, midLineValue:0, lineWidth:0.95}).replace('class="mini-chart"', 'style="width:100%;height:180px;background:rgba(0,0,0,.2);border-radius:10px"');
  let html = `<div class="section"><h3>⚖ 法律 / 体制</h3>
    <div class="headline-card">
      <div class="meta">現在の体制軸</div>
      <div class="body"><strong style="color:${summary.color}">${summary.title}</strong><br>${summary.body}<br>現在値 ${Math.round(S.ideologyScore)} / 100</div>
    </div>
    <div class="large-chart-wrap" style="margin-top:8px">${chart}</div>
    <div class="history-legend"><span>上ほど民主主義</span><span>下ほど社会主義・統制</span><span><button class="btn btn-sm" onclick="openHistoryExplorer('ideology')">履歴を開く</button></span></div>
    <div class="compact-note" style="margin-top:8px">${S.lawChangeIntervalYears}年ごとに法律を変更できます。${getLawCooldownLabel()}。体制の変化は外交・企業収益・投資家評価に強く反映されます。</div>
  </div>
  <div class="section"><h3>📜 制定できる法律</h3>`;
  getLawCatalog().forEach(law => {
    const previewScore = law.setMode === 'toward_zero' ? Math.round(lerp(S.ideologyScore, 0, 0.55)) : clamp(S.ideologyScore + law.shift, -100, 100);
    html += `<div class="work-card">
      <div class="work-row">
        <div>
          <div style="font-size:14px;font-weight:700">${law.name}</div>
          <div class="work-meta">${law.desc}<br>制定後の体制軸: ${previewScore}</div>
        </div>
        <div class="save-actions">
          <span class="pill">💰${formatMoneyCompact(law.cost)} / 🏛${law.pol}</span>
          <button class="btn btn-sm btn-blue" onclick="enactLaw('${law.id}')" ${locked ? 'disabled' : ''}>制定</button>
        </div>
      </div>
    </div>`;
  });
  html += `</div><div class="section"><h3>🌍 外交への影響</h3>`;
  tradePartners.forEach(partner => {
    const pref = getCountryIdeologyPreference(partner.id);
    const alignment = 100 - Math.abs(S.ideologyScore - pref * 100);
    html += `<div class="trade-item"><span>${partner.name}</span><span>相性 ${Math.round(alignment)} / 友好 ${Math.round(partner.relation)}</span></div>`;
  });
  html += '</div>';
  return html;
}

function renderGoals(){
  const goals = getGoalDefinitions();
  const completed = goals.filter(goal => goal.done).length;
  const advisorNotes = getAdvisorNotes();
  let html = `<div class="section"><h3>🎯 国家プロジェクト</h3>
    <div class="headline-card ${completed >= 4 ? 'good' : completed === 0 ? 'bad' : ''}">
      <div class="meta">進捗サマリー</div>
      <div class="body"><strong>${completed} / ${goals.length} 完了</strong><br>国家スコア ${S.nationalScore} | ポートフォリオ ${formatMoneyCompact(Math.round(getPortfolioValue()))} | 平均友好 ${getAverageRelation().toFixed(0)}</div>
    </div>`;
  goals.forEach(goal => {
    const pct = Math.round(goal.progress * 100);
    html += `<div class="goal-card ${goal.done ? 'done' : ''}">
      <div class="goal-title"><span>${goal.title}</span><span>${goal.done ? '達成' : `${pct}%`}</span></div>
      <div class="goal-desc">${goal.desc}<br>${goal.reward}</div>
      <div class="goal-progress"><div class="fill" style="width:${pct}%"></div></div>
      <div style="font-size:8px;color:#ccc">${goal.detail}</div>
    </div>`;
  });
  html += '</div><div class="section"><h3>🧭 政策アドバイス</h3>';
  advisorNotes.forEach(note => {
    html += `<div class="advisor-item ${note.type === 'danger' ? 'danger' : note.type === 'warn' ? 'warn' : ''}">${note.text}</div>`;
  });
  html += '</div>';
  return html;
}

const PANEL_EN_REPLACEMENTS = [
  ['国家ヘッドライン','National Headlines'],
  ['ニュース履歴','News Archive'],
  ['国家プロジェクト','National Projects'],
  ['政策アドバイス','Policy Advice'],
  ['研究開発','Research & Development'],
  ['国際比較','International Comparison'],
  ['各国比較','Country Comparison'],
  ['国内資源','Domestic Resources'],
  ['年間予算','Annual Budget'],
  ['貿易設定','Trade Setup'],
  ['貿易一覧','Trade Ledger'],
  ['地域開発','Regional Development'],
  ['日本銀行','Central Bank'],
  ['公共事業','Public Works'],
  ['サイバー戦力','Cyber Operations'],
  ['災害管制','Disaster Control'],
  ['宇宙開発局','Space Development Bureau'],
  ['世界市場 / 外国株','World Market / Foreign Stocks'],
  ['外国企業','Foreign Companies'],
  ['世界AIログ','World AI Log'],
  ['会社技術方針','Company Tech Policy'],
  ['技術方針まとめ','Tech Strategy Summary'],
  ['株式市場','Stock Market'],
  ['保有ポートフォリオ','Portfolio'],
  ['国家見通し','National Outlook'],
  ['進捗サマリー','Progress Summary'],
  ['現在の体制軸','Current Ideology Axis'],
  ['制定できる法律','Available Laws'],
  ['外交への影響','Diplomatic Impact'],
  ['履歴を開く','Open History'],
  ['制定後の体制軸','Post-enactment Axis'],
  ['制定','Enact'],
  ['相性','Alignment'],
  ['友好','Relations'],
  ['完了','Completed'],
  ['達成','Achieved'],
  ['電力需給','Power Balance'],
  ['主資源','Key Resources'],
  ['災害状況','Disaster Status'],
  ['自然災害継続中','Active Natural Disaster'],
  ['全期間','All Time'],
  ['全て','All'],
  ['価格順','By Price'],
  ['変動順','By Change'],
  ['売上順','By Revenue'],
  ['総額:','Total:'],
  ['標準開発','Standard R&D'],
  ['次の節目','Next Milestone'],
  ['ロードマップ達成済み','Roadmap Completed'],
  ['詳細','Details'],
  ['発電量','Power Output'],
  ['需要','Demand'],
  ['供給国','Suppliers'],
  ['輸入','Imports'],
  ['輸出','Exports'],
  ['国債残高','Government Debt'],
  ['週次利払い','Weekly Interest'],
  ['週次元本返済','Weekly Principal'],
  ['税収','Tax Revenue'],
  ['予算支出','Budget Spend'],
  ['貿易収支','Trade Balance'],
  ['宇宙収入','Space Revenue'],
  ['公共収入','Public Revenue'],
  ['公共運転費','Public Operating Cost'],
  ['元本返済','Principal Repayment'],
  ['企業支援','Company Support'],
  ['利払い','Interest Payments'],
  ['平常','Normal'],
  ['資金不足','Not enough funds'],
  ['政治資本不足','Not enough political capital'],
];

function translatePanelMarkup(html){
  if(S.language !== 'en' || !html) return html;
  let out = String(html);
  PANEL_EN_REPLACEMENTS.forEach(([ja, en]) => {
    out = out.split(ja).join(en);
  });
  return out;
}

function renderPanel(){
  const content = document.getElementById('panelContent');
  if(!content) return;
  setPanelTabActive(S.currentPanel);
  const renderers = {
    market:renderMarket,
    news:renderNews,
    goals:renderGoals,
    research:renderResearch,
    world:renderWorld,
    foreign:renderForeignMarket,
    compare:renderCompare,
    resources:renderResources,
    policy:renderPolicy,
    trade:renderTrade,
    develop:renderDevelop,
    bank:renderBank,
    law:renderLaw,
    infra:renderInfra,
    cyber:renderCyber,
    chaos:renderChaos,
    space:renderSpace,
    nuke:renderNuke,
  };
  const panelKey = renderers[S.currentPanel] ? S.currentPanel : 'market';
  try{
    content.innerHTML = translatePanelMarkup(renderers[panelKey]());
  } catch(error){
    reportRuntimeError(`panel:${panelKey}`, error);
    content.innerHTML = `<div class="section"><h3>${S.language === 'en' ? '⚠ Panel Load Error' : '⚠ パネル読込エラー'}</h3><div class="compact-note">${escapeHtml(error?.message || error)}</div></div>`;
  }
}

function renderCompanyTechFocusCard(){
  const company = getCompanyByName(S.techFocusCompany) || companies[0];
  if(!company) return '';
  S.techFocusCompany = company.name;
  syncCompanyRoadmap(company);
  const sector = sectors[company.sector];
  const activeTrack = getCompanyTrack(company);
  const stage = company.roadmap?.[company.roadmapIndex];
  return `<div class="section">
    <h3>🧠 会社技術方針</h3>
    <div class="trade-item" style="align-items:flex-start">
      <span style="display:flex;flex-direction:column;align-items:flex-start;gap:2px">
        <strong>${sector?.icon || '🏢'} ${company.name}</strong>
        <span class="compact-note">${activeTrack ? activeTrack.name : '標準開発'} / 技術深度 ${Math.round(company.techDepth)} / 進捗 ${Math.round(company.techProgress)}</span>
      </span>
      <button class="btn btn-sm btn-blue" onclick="openCompanyDetail('${company.name}')">詳細</button>
    </div>
    <div class="compact-note" style="margin-top:8px">${activeTrack ? activeTrack.desc : '標準開発では会社本来のロードマップを進めます。選んでいる方針だけが進行し、途中で変えても進捗は保存されます。'}</div>
    <div class="save-actions" style="margin-top:8px">
      <button class="btn btn-sm ${!company.selectedTechTrack?'active':''}" onclick="setFocusedCompanyTechTrack('${company.name}','')">標準</button>
      ${(company.techTracks || []).map(track => `<button class="btn btn-sm ${company.selectedTechTrack===track.id?'active':''}" onclick="setFocusedCompanyTechTrack('${company.name}','${track.id}')">${track.name}</button>`).join('')}
    </div>
    ${stage ? `<div class="headline-card" style="margin-top:8px"><div class="meta">次の節目</div><div class="body">${stage.name}<br>${stage.note}<br>必要進捗 ${stage.threshold}</div></div>` : '<div class="headline-card good" style="margin-top:8px"><div class="meta">ロードマップ</div><div class="body">この会社のロードマップは全段階達成済みです。</div></div>'}
  </div>`;
}

function renderTechPolicySummary(filteredCompanies){
  const targets = (filteredCompanies && filteredCompanies.length ? filteredCompanies : companies).slice(0, 8);
  if(!targets.length) return '';
  return `<div class="section"><h3>🧭 技術方針まとめ</h3>
    <div class="compact-note">カテゴリ別の絞り込みに連動します。ここから各社の現在方針と次の節目を一気に確認して切り替えられます。</div>
    ${targets.map(company => {
      syncCompanyRoadmap(company);
      const activeTrack = getCompanyTrack(company);
      const stage = company.roadmap?.[company.roadmapIndex];
      const sector = sectors[company.sector];
      return `<div class="work-card">
        <div class="work-row">
          <div>
            <div style="font-size:14px;font-weight:700">${sector?.icon || '🏢'} ${company.name}</div>
            <div class="work-meta">${activeTrack ? activeTrack.name : '標準開発'} / 技術深度 ${Math.round(company.techDepth)} / 進捗 ${Math.round(company.techProgress)}${stage ? `<br>次の節目: ${stage.name}` : '<br>ロードマップ達成済み'}</div>
          </div>
          <div class="save-actions">
            <button class="btn btn-sm ${!company.selectedTechTrack ? 'active' : ''}" onclick="setFocusedCompanyTechTrack('${company.name}','')">標準</button>
            ${(company.techTracks || []).map(track => `<button class="btn btn-sm ${company.selectedTechTrack===track.id ? 'active' : ''}" onclick="setFocusedCompanyTechTrack('${company.name}','${track.id}')">${track.name}</button>`).join('')}
            <button class="btn btn-sm btn-blue" onclick="S.techFocusCompany='${company.name}'; openCompanyDetail('${company.name}')">詳細</button>
          </div>
        </div>
      </div>`;
    }).join('')}
  </div>`;
}

function renderMarket(){
  let filtered = S.companySectorFilter === 'all' ? [...companies] : companies.filter(c=>c.sector===S.companySectorFilter);
  if(S.companySortMode === 'price') filtered.sort((a,b)=>b.price-a.price);
  else if(S.companySortMode === 'revenue') filtered.sort((a,b)=>b.revenue-a.revenue);
  else filtered.sort((a,b)=>Math.abs(b.price/b.prevPrice-1)-Math.abs(a.price/a.prevPrice-1));
  if(filtered.length && S.companySectorFilter !== 'all' && !filtered.some(company => company.name === S.techFocusCompany)){
    S.techFocusCompany = filtered[0].name;
  }
  let html = renderCompanyTechFocusCard();
  html += '<div class="section"><h3>📊 株式市場</h3>';
  html += `<div style="display:flex;gap:3px;margin-bottom:4px;flex-wrap:wrap">
    <button class="btn btn-sm ${S.companySectorFilter==='all'?'active':''}" onclick="S.companySectorFilter='all';renderPanel()">全て</button>
    ${Object.entries(sectors).map(([k,s])=>
      `<button class="btn btn-sm market-filter-btn ${S.companySectorFilter===k?'active':''}" style="border-color:${s.color}" data-tip="${s.name}" 
        onclick="S.companySectorFilter='${k}';renderPanel()">${s.icon}</button>`
    ).join('')}
    <span style="margin-left:auto"></span>
    <button class="btn btn-sm ${S.companySortMode==='price'?'active':''}" onclick="S.companySortMode='price';renderPanel()">価格順</button>
    <button class="btn btn-sm ${S.companySortMode==='change'?'active':''}" onclick="S.companySortMode='change';renderPanel()">変動順</button>
    <button class="btn btn-sm ${S.companySortMode==='revenue'?'active':''}" onclick="S.companySortMode='revenue';renderPanel()">売上順</button>
  </div>`;
  filtered.forEach(c => {
    const sec = sectors[c.sector];
    const ch = ((c.price - c.prevPrice) / c.prevPrice * 100).toFixed(1);
    const up = c.price >= c.prevPrice;
    const owned = S.ownedStocks[c.name] || 0;
    const support = getCompanyAnnualSupport(c.name);
    const ownedRatio = clamp(owned / Math.max(1, c.sharesOutstanding) * 100, 0, 100);
    const roadmapStage = c.roadmap?.[c.roadmapIndex];
    html += `<div class="company" style="border-left-color:${sec.color}" onclick="S.techFocusCompany='${c.name}'; openCompanyDetail('${c.name}')">
      <div class="c-left" style="display:flex;flex-direction:column;align-items:flex-start">
        <span class="c-name">${sec.icon}${c.name}${owned>0?` 🔷${ownedRatio.toFixed(1)}%`:''}</span>
        <span class="compact-note">売上 ¥${Math.round(c.revenue).toLocaleString()} / 好感度 ${Math.round(c.favorability)} / 技術 ${Math.round(c.techDepth)}${support>0 ? ` / 年間支援 ${formatMoneyCompact(support)}` : ''}${roadmapStage ? ` / 次 ${roadmapStage.name}` : ''}</span>
      </div>
      <div class="c-right">
        <span class="c-change" style="color:${up?'#4ecca3':'#e94560'}">${up?'▲':'▼'}${ch}%</span>
        <span class="c-price" style="color:${up?'#4ecca3':'#e94560'}">${formatMoneyCompact(c.price)}</span>
      </div>
    </div>`;
  });
  html += '</div>';
  
  // Portfolio summary
  const totalStockValue = Object.entries(S.ownedStocks).reduce((s,[name,qty])=>{
    const c = companies.find(co=>co.name===name);
    return s + (c ? c.price * qty : 0);
  }, 0);
  if(totalStockValue > 0){
    html += `<div class="section"><h3>💼 保有ポートフォリオ</h3>
      <div style="font-size:11px;font-weight:bold;color:#4ecca3">総額: ${formatMoneyCompact(totalStockValue)}</div>`;
    Object.entries(S.ownedStocks).forEach(([name,qty])=>{
      const c = companies.find(co=>co.name===name);
      if(!c) return;
      html += `<div class="trade-item"><span>${name} ×${qty}</span><span>${formatMoneyCompact(c.price*qty)}</span></div>`;
    });
    html += '</div>';
  }
  html += renderTechPolicySummary(filtered);
  return html;
}

function renderResources(){
  let html = '<div class="section"><h3>📦 国内資源</h3>';
  if(S.discoveredMaterials.length){
    html += `<div class="headline-card good"><div class="meta">新規発見資源</div><div class="body">${[...new Set(S.discoveredMaterials.map(entry => resources[entry.id]?.name).filter(Boolean))].join(' / ')}</div></div>`;
  }
  Object.entries(resources).sort((a, b) => {
    const aRatio = a[1].max ? a[1].amount / a[1].max : 1;
    const bRatio = b[1].max ? b[1].amount / b[1].max : 1;
    return aRatio - bRatio;
  }).forEach(([k,r])=>{
    const pct = r.max > 0 ? Math.min(100, r.amount / r.max * 100) : 0;
    const isZero = r.amount <= 0;
    html += `<div class="resource-bar${isZero?' resource-zero':''}">
      <span class="r-name">${r.name}${r.domestic?'':'🌐'}</span>
      <div class="r-bar"><div class="r-fill" style="width:${pct}%;background:${r.color}"></div></div>
      <span class="r-val">${formatAmount(r.amount)}${r.unit?'/'+r.unit:''}</span>
      ${tradePartners.some(partner => partner.sells.includes(k)) ? `<button class="resource-jump-btn" onclick="jumpTradeResource('${k}')">貿易へ</button>` : ''}
    </div>`;
  });
  // Discovered unique materials
  S.discoveredMaterials.forEach(dm=>{
    const mat = [...uniqueMaterials, ...spaceMaterials].find(m=>m.id===dm.id);
    const r = resources[dm.id];
    if(!mat || !r) return;
    html += `<div class="resource-bar" style="border-left:2px solid ${mat.color};padding-left:4px">
      <span class="r-name" style="color:${mat.color}">✦${mat.name}</span>
      <div class="r-bar"><div class="r-fill" style="width:${Math.min(100,r.amount/1000*100)}%;background:${mat.color}"></div></div>
      <span class="r-val">${formatAmount(r.amount)}</span>
    </div>`;
  });
  html += '</div>';
  return html;
}

window.jumpTradeResource = function(resourceKey){
  S.tradeFocusResource = resourceKey;
  S.currentPanel = 'trade';
  S.currentMapMode = 'japan';
  loadViewForMode('japan');
  renderPanel();
  updateMapHud();
  drawMap();
};

window.openTradeForResource = function(resourceKey){
  jumpTradeResource(resourceKey);
  closeModal();
  setMapFocus('panel');
};

function formatAmount(n){
  if(n >= 1000000) return (n/1000000).toFixed(1)+'M';
  if(n >= 1000) return (n/1000).toFixed(1)+'K';
  return n.toFixed(0);
}

// ============================================================
// POLICY
// ============================================================
function renderPolicy(){
  const locked = S.budgetSetYear === S.year;
  const targetOptions = [''].concat(companies.map(company => company.name));
  const supportRows = normalizeBudgetSupportList(S.budgetDraft.companySupports, S.budgetDraft.company, S.budgetDraft.companySupport);
  const supportTotal = supportRows.reduce((sum, row) => sum + row.amount, 0);
  return `<div class="section"><h3>💰 年間予算</h3>
    <p class="compact-note">予算は年1回だけ設定できます。設定済み年: ${S.budgetSetYear === S.year ? `${S.year}年` : '未設定'}。軍事・教育・技術に加えて、複数企業への年間支援と税率もここでまとめて決めます。</p>
    <div class="slider-row">
      <label>⚔軍事</label>
      <input type="range" min="0" max="50000" step="500" value="${S.budgetDraft.military}" oninput="setBudgetDraft('military', this.value)">
      <span class="sv" id="budgetMilitaryVal">${Math.round(S.budgetDraft.military)}</span>
    </div>
    <div class="slider-row">
      <label>📚教育</label>
      <input type="range" min="0" max="50000" step="500" value="${S.budgetDraft.education}" oninput="setBudgetDraft('education', this.value)">
      <span class="sv" id="budgetEducationVal">${Math.round(S.budgetDraft.education)}</span>
    </div>
    <div class="slider-row">
      <label>🔬技術</label>
      <input type="range" min="0" max="50000" step="500" value="${S.budgetDraft.technology}" oninput="setBudgetDraft('technology', this.value)">
      <span class="sv" id="budgetTechnologyVal">${Math.round(S.budgetDraft.technology)}</span>
    </div>
    <div class="slider-row">
      <label>🧾税率</label>
      <input type="range" min="5" max="35" step="1" value="${S.taxRate}" oninput="setTaxDraft(this.value)">
      <span class="sv" id="budgetTaxVal">${Math.round(S.taxRate)}%</span>
    </div>
    <div class="section" style="margin-top:10px">
      <h3>🏢 年間の企業支援</h3>
      <div class="compact-note">複数社に別々の金額を設定できます。支援中の会社は市場と価格バーでも分かるように表示されます。現在合計: ${formatMoneyCompact(supportTotal)}</div>
      ${supportRows.map((row, index) => `<div class="support-row">
        <div class="slider-row">
          <label>支援先 ${index + 1}</label>
          <select class="btn" style="flex:1;background:#0f1830" onchange="setBudgetSupportCompany(${index}, this.value)">
            ${targetOptions.map(name => `<option value="${name}" ${row.company===name?'selected':''}>${name || '未選択'}</option>`).join('')}
          </select>
          <button class="btn btn-sm" onclick="removeBudgetSupportRow(${index})" ${supportRows.length <= 2 ? 'disabled' : ''}>削除</button>
        </div>
        <div class="slider-row">
          <label>年間支援額</label>
          <input type="range" min="0" max="50000" step="500" value="${row.amount}" oninput="setBudgetSupportAmount(${index}, this.value); this.nextElementSibling.textContent = Math.round(this.value)">
          <span class="sv">${Math.round(row.amount)}</span>
        </div>
      </div>`).join('')}
      <div class="save-actions">
        <button class="btn btn-blue" onclick="addBudgetSupportRow()">支援先を追加</button>
      </div>
    </div>
    <div class="large-chart-wrap">
      <div class="compact-note">
        軍事: 防衛力、防衛株、核・宇宙成功率に寄与<br>
        教育: 支持率、雇用、人口、長期GDP、研究条件に寄与<br>
        技術: 技術力、IT/宇宙株、サイバー、研究成功率に寄与<br>
        税率: 増税は税収とバブル抑制に寄与する一方、景気と支持率を圧迫します<br>
        企業支援: 指定企業に継続的な売上押上げ。企業好感度と技術進展にも効きます
      </div>
    </div>
    <div class="save-actions">
      <button class="btn btn-green" onclick="applyAnnualBudget()" ${locked ? 'disabled' : ''}>${locked ? '今年は設定済み' : 'この内容で今年の予算を確定'}</button>
      <button class="btn btn-blue" onclick="resetBudgetDraft()">ドラフトを戻す</button>
    </div>
  </div>`;
}

window.setBudgetDraft = function(field, value){
  S.budgetDraft[field] = +value;
  const map = {
    military:'budgetMilitaryVal',
    education:'budgetEducationVal',
    technology:'budgetTechnologyVal',
  };
  const element = document.getElementById(map[field]);
  if(element) element.textContent = Math.round(+value);
};

window.setBudgetCompany = function(name){
  S.budgetDraft.company = name;
};

window.setTaxDraft = function(value){
  S.taxRate = +value;
  const element = document.getElementById('budgetTaxVal');
  if(element) element.textContent = `${Math.round(S.taxRate)}%`;
};

window.addBudgetSupportRow = function(){
  S.budgetDraft.companySupports = normalizeBudgetSupportList(S.budgetDraft.companySupports, S.budgetDraft.company, S.budgetDraft.companySupport);
  if(S.budgetDraft.companySupports.length < 6) S.budgetDraft.companySupports.push({company:'', amount:0});
  openAction('budget');
};

window.removeBudgetSupportRow = function(index){
  S.budgetDraft.companySupports = normalizeBudgetSupportList(S.budgetDraft.companySupports, S.budgetDraft.company, S.budgetDraft.companySupport);
  S.budgetDraft.companySupports.splice(index, 1);
  openAction('budget');
};

window.setBudgetSupportCompany = function(index, companyName){
  S.budgetDraft.companySupports = normalizeBudgetSupportList(S.budgetDraft.companySupports, S.budgetDraft.company, S.budgetDraft.companySupport);
  if(!S.budgetDraft.companySupports[index]) S.budgetDraft.companySupports[index] = {company:'', amount:0};
  S.budgetDraft.companySupports[index].company = companyName;
};

window.setBudgetSupportAmount = function(index, value){
  S.budgetDraft.companySupports = normalizeBudgetSupportList(S.budgetDraft.companySupports, S.budgetDraft.company, S.budgetDraft.companySupport);
  if(!S.budgetDraft.companySupports[index]) S.budgetDraft.companySupports[index] = {company:'', amount:0};
  S.budgetDraft.companySupports[index].amount = +value;
};

window.resetBudgetDraft = function(){
  S.budgetDraft = JSON.parse(JSON.stringify(S.annualBudget));
  openAction('budget');
};

window.applyAnnualBudget = function(){
  if(S.budgetSetYear === S.year){ addEvent('❌ 今年の予算はすでに確定済みです', 'bad'); return; }
  S.budgetDraft.companySupports = normalizeBudgetSupportList(S.budgetDraft.companySupports, S.budgetDraft.company, S.budgetDraft.companySupport);
  S.budgetDraft.company = S.budgetDraft.companySupports[0]?.company || '';
  S.budgetDraft.companySupport = S.budgetDraft.companySupports[0]?.amount || 0;
  S.annualBudget = JSON.parse(JSON.stringify(S.budgetDraft));
  S.budgetSetYear = S.year;
  S.militaryBudget = S.annualBudget.military / 2500;
  S.educationBudget = S.annualBudget.education / 2500;
  S.techBudget = S.annualBudget.technology / 2500;
  S.annualBudget.companySupports = normalizeBudgetSupportList(S.annualBudget.companySupports, S.annualBudget.company, S.annualBudget.companySupport);
  addEvent(`💰 ${S.year}年の年間予算を確定`, 'good');
  openAction('budget');
};

// ============================================================
// TRADE (import + export)
// ============================================================
function renderTrade(){
  const draftMode = S.tradeDraftMode === 'amount' ? 'amount' : 'percent';
  const draftLabel = draftMode === 'amount'
    ? `${formatTradeQuantity(S.tradeAmountDraft)} / 便`
    : `${S.tradeQuantityDraft}% / 便`;
  const shortageResources = Object.entries(resources)
    .filter(([key, resource]) => resource.max && resource.amount / resource.max < 0.15 && tradePartners.some(partner => partner.sells.includes(key)))
    .sort((a, b) => (a[1].amount / a[1].max) - (b[1].amount / b[1].max));
  const productKeys = [...new Set([
    ...Object.keys(resources).filter(key => resources[key].price),
    ...Object.keys(productCatalog).filter(hasUnlockedProduct),
    ...tradePartners.flatMap(partner => partner.buys.filter(key => !resources[key] && (!productCatalog[key] || hasUnlockedProduct(key)))),
    ...tradePartners.flatMap(partner => partner.sells.filter(key => !resources[key] && (!productCatalog[key] || hasUnlockedProduct(key)))),
  ])];
  const orderedProductKeys = [...productKeys].sort((a, b) => {
    if(S.tradeFocusResource && a === S.tradeFocusResource) return -1;
    if(S.tradeFocusResource && b === S.tradeFocusResource) return 1;
    const aShort = shortageResources.some(([key]) => key === a) ? 1 : 0;
    const bShort = shortageResources.some(([key]) => key === b) ? 1 : 0;
    return bShort - aShort;
  });
  let html = `<div class="section"><h3>🚢 製品別貿易</h3>
    <p class="compact-note">先に取引量を決めてから相手国を選ぶ方式です。契約後は輸送船団が日本と各国を往復し、到着まで数週間かかります。現在の指定量: ${draftLabel}</p>
    <div class="trade-slider-wrap">
      <div class="trade-slider-head">
        <span>${S.language === 'en' ? 'Import / Export Share Per Shipment' : '輸入・輸出の便ごとの比率'}</span>
        <span class="trade-slider-value" id="tradeDraftValue">${draftLabel}</span>
      </div>
      <div class="save-actions" style="margin-top:6px">
        <button class="btn btn-sm ${draftMode==='percent'?'active':''}" onclick="setTradeDraftMode('percent')">${S.language === 'en' ? 'Percent' : 'パーセント'}</button>
        <button class="btn btn-sm ${draftMode==='amount'?'active':''}" onclick="setTradeDraftMode('amount')">${S.language === 'en' ? 'Amount' : '数量指定'}</button>
      </div>
      ${draftMode === 'percent'
        ? `<input id="tradeDraftSlider" type="range" min="1" max="100" step="1" value="${S.tradeQuantityDraft}" oninput="setTradeDraft(this.value)" onchange="setTradeDraft(this.value)">`
        : `<div class="save-actions" style="margin-top:10px">
            <input id="tradeAmountInput" type="number" min="1" step="10" value="${Math.round(S.tradeAmountDraft || 1)}" onchange="setTradeAmountDraft(this.value)" style="width:180px;padding:8px;background:#0f1530;color:#fff;border:1px solid rgba(255,255,255,.08);border-radius:8px">
            <span class="compact-note">${S.language === 'en' ? 'Direct quantity per shipment' : '1便ごとの欲しい量を直接指定'}</span>
          </div>`}
      <div class="compact-note" style="margin-top:8px">${draftMode === 'percent'
        ? (S.language === 'en' ? 'Use the slider to fine-tune shipment size before choosing a partner.' : 'スライダーで便の大きさを細かく決めてから相手国を選べます。')
        : (S.language === 'en' ? 'Use a direct quantity if you already know how much stock you want to move.' : '必要量が分かっている時は、数量指定でそのまま便ごとに注文できます。')}</div>
    </div>
    ${shortageResources.length ? `<div class="section" style="margin-top:8px"><h3>⚠ 不足している資源</h3>${shortageResources.map(([key, resource]) => `<div class="trade-item"><span>${resource.name}</span><span>${Math.round(resource.amount / resource.max * 100)}%</span><button class="btn btn-sm btn-yellow" onclick="jumpTradeResource('${key}')">候補へ</button></div>`).join('')}</div>` : ''}
    ${S.tradeFocusResource ? `<div class="status-badge warn">注目資源: ${resources[S.tradeFocusResource]?.name || S.tradeFocusResource} <button class="btn btn-sm" style="margin-left:6px" onclick="S.tradeFocusResource=''; renderPanel();">解除</button></div>` : ''}
    <div class="section" style="margin-top:8px;min-height:168px"><h3>⛴ 輸送中の便</h3>${S.tradeShipments.length ? S.tradeShipments.slice(0, 12).map(shipment => `<div class="trade-item"><span>${shipment.isSell ? '輸出' : '輸入'} ${tradePartners[shipment.partnerIdx].name} / ${shipment.resourceName}</span><span>残り ${shipment.weeksLeft}週</span></div>`).join('') : '<div class="compact-note">現在航行中の便はありません。</div>'}</div>`;
  orderedProductKeys.forEach(key => {
    const product = getExportableResource(key) || {name:resources[key]?.name || key, price:resources[key]?.price || 100, resource:key};
    const imports = tradePartners.map((partner, partnerIdx) => ({partner, partnerIdx})).filter(entry => entry.partner.sells.includes(key));
    const exports = tradePartners.map((partner, partnerIdx) => ({partner, partnerIdx})).filter(entry => entry.partner.buys.includes(key));
    if(!imports.length && !exports.length) return;
    const active = S.tradeDeals.filter(deal => deal.resource === key);
    const focused = S.tradeFocusResource === key;
    html += `<div class="goal-card" style="${focused ? 'box-shadow:0 0 0 1px rgba(255,193,86,.35) inset;' : ''}">
      <div class="goal-title"><span>${product.name}</span><span>${resources[key] ? `${formatAmount(resources[key].amount)}${resources[key].unit?'/'+resources[key].unit:''}` : '輸出専用品'}</span></div>
      <div class="goal-desc">輸入 ${imports.length}ルート / 輸出 ${exports.length}ルート / 現在輸送中 ${S.tradeShipments.filter(shipment => shipment.resourceKey === key).length}便</div>
      ${active.length ? active.map(deal => {
        const dispatchInterval = Math.max(2, Math.round((deal.travelWeeks || 2) * 0.55));
        const monthlyValue = Math.abs(deal.cost || 0) * (4 / dispatchInterval);
        return `<div class="trade-item"><span>${deal.isSell?'⬆':'⬇'} ${tradePartners[deal.partnerIdx].name} ${formatTradeDealSpec(deal)} / 到着 ${deal.travelWeeks}週</span><span>${deal.isSell?'+':'-'}${formatMoneyCompact(Math.round(Math.abs(deal.cost || 0)))} / 便<br><span class="compact-note">月目安 ${deal.isSell?'+':'-'}${formatMoneyCompact(Math.round(monthlyValue))} / ${dispatchInterval}週ごと</span></span><span><button class="btn btn-sm" onclick="adjustTradeDeal(${S.tradeDeals.indexOf(deal)},-1)">-</button> <button class="btn btn-sm" onclick="adjustTradeDeal(${S.tradeDeals.indexOf(deal)},1)">+</button> <button class="btn btn-sm" onclick="stopTradeDeal(${S.tradeDeals.indexOf(deal)})">✕</button></span></div>`;
      }).join('') : '<div class="compact-note">取引なし</div>'}
      <div style="margin-top:6px">
        <div class="compact-note">輸入先</div>
        <div style="display:flex;flex-wrap:wrap;gap:4px;margin-top:4px">${imports.map(entry => {
          const quote = getTradeDealPreview(entry.partnerIdx, key, false, S.tradeQuantityDraft, null, null, draftMode, S.tradeAmountDraft);
          return `<button class="btn btn-sm btn-yellow" onclick="startImport(${entry.partnerIdx},'${key}',${S.tradeQuantityDraft})">${entry.partner.name}<br><span style="font-size:10px;color:#ffd180">${formatTradePreviewLabel(quote, false)} / ${quote?.travelWeeks || '-'}週</span></button>`;
        }).join('') || '<span class="compact-note">なし</span>'}</div>
      </div>
      <div style="margin-top:6px">
        <div class="compact-note">輸出先</div>
        <div style="display:flex;flex-wrap:wrap;gap:4px;margin-top:4px">${exports.map(entry => {
          const quote = getTradeDealPreview(entry.partnerIdx, key, true, S.tradeQuantityDraft, null, null, draftMode, S.tradeAmountDraft);
          return `<button class="btn btn-sm btn-green" onclick="startExport(${entry.partnerIdx},'${key}',${S.tradeQuantityDraft})">${entry.partner.name}<br><span style="font-size:10px;color:#baf5d1">${formatTradePreviewLabel(quote, true)} / ${quote?.travelWeeks || '-'}週</span></button>`;
        }).join('') || '<span class="compact-note">なし</span>'}</div>
      </div>
    </div>`;
  });
  html += '</div>';
  return html;
}

function getExportableResource(key){
  // Map abstract keys to actual resources
  if(productCatalog[key]){
    if(!hasUnlockedProduct(key)) return null;
    return {name:productCatalog[key].name, price:productCatalog[key].price, resource:productCatalog[key].resource};
  }
  if(resources[key]) return {name:resources[key].name, price:resources[key].price||100, resource:key};
  return null;
}

function formatTradeQuantity(amount, unit=''){
  return `${formatAmount(Math.max(1, amount))}${unit ? '/' + unit : ''}`;
}

function formatTradeDealSpec(deal){
  if((deal.mode || 'percent') === 'amount'){
    return formatTradeQuantity(deal.amountDraft || deal.amount || 0, deal.unit || '');
  }
  return `${deal.pct}%`;
}

function formatTradePreviewLabel(quote, isSell=false){
  if(!quote) return isSell ? '+0' : '-0';
  return `${isSell?'+':'-'}${formatMoneyCompact(Math.round(quote.cost || 0))} / ${formatTradeQuantity(quote.amount || 0, quote.unit || '')}`;
}

function getTradeDealPreview(partnerIdx, key, isSell=false, pct=S.tradeQuantityDraft, actualResource=null, priceBaseOverride=null, mode=S.tradeDraftMode, quantityOverride=S.tradeAmountDraft){
  const partner = tradePartners[partnerIdx];
  if(!partner) return null;
  const exportable = getExportableResource(key);
  const actualResourceKey = actualResource || exportable?.resource || key;
  const resource = resources[actualResourceKey];
  const cyberAdv = S.cyberPower > partner.cyberPower;
  const exchange = getTradeExchangeFactor(partner);
  const priceBase = priceBaseOverride || exportable?.price || resource?.price || 100;
  const draftMode = mode === 'amount' ? 'amount' : 'percent';
  const amount = draftMode === 'amount'
    ? Math.max(1, Number(quantityOverride) || 1)
    : Math.max(1, (resource?.max || 100000) * (pct / 100) * (isSell ? 0.018 : 0.032));
  const travelWeeks = getCountryShippingWeeks(partnerIdx);
  const dispatchInterval = Math.max(2, Math.round(travelWeeks * 0.55));
  const cost = isSell
    ? (priceBase * amount * 0.01 * exchange * (cyberAdv ? 1.08 : 0.98) * clamp(partner.relation / 60, 0.4, 1.6))
    : (priceBase * amount * 0.01 * exchange * (cyberAdv ? 0.94 : 1.08) * clamp((120 - partner.relation) / 65, 0.5, 1.8));
  return {
    partner,
    amount,
    unit: resource?.unit || '',
    cost,
    travelWeeks,
    dispatchInterval,
    monthlyValue: cost * (4 / dispatchInterval),
    mode:draftMode,
  };
}

window.setTradeDraft = function(pct){
  S.tradeQuantityDraft = clamp(Math.round(Number(pct) || 1), 1, 100);
  const valueEl = document.getElementById('tradeDraftValue');
  if(valueEl) valueEl.textContent = `${S.tradeQuantityDraft}% / 便`;
  const sliderEl = document.getElementById('tradeDraftSlider');
  if(sliderEl && Number(sliderEl.value) !== S.tradeQuantityDraft) sliderEl.value = S.tradeQuantityDraft;
};

window.setTradeDraftMode = function(mode){
  S.tradeDraftMode = mode === 'amount' ? 'amount' : 'percent';
  if(S.currentPanel === 'trade') renderPanel();
};

window.setTradeAmountDraft = function(amount){
  S.tradeAmountDraft = Math.max(1, Math.round(Number(amount) || 1));
  const valueEl = document.getElementById('tradeDraftValue');
  if(valueEl) valueEl.textContent = `${formatTradeQuantity(S.tradeAmountDraft)} / 便`;
  const inputEl = document.getElementById('tradeAmountInput');
  if(inputEl && Number(inputEl.value) !== S.tradeAmountDraft) inputEl.value = S.tradeAmountDraft;
};

function recalcTradeDeal(deal){
  const preview = getTradeDealPreview(deal.partnerIdx, deal.resource, !!deal.isSell, deal.pct, deal.actualResource, deal.priceBase, deal.mode, deal.amountDraft);
  if(!preview) return;
  deal.travelWeeks = preview.travelWeeks;
  deal.amount = preview.amount;
  deal.unit = preview.unit;
  deal.cost = preview.cost;
}

window.startImport = function(pi, res, pct=S.tradeQuantityDraft){
  const r = resources[res]; if(!r) return;
  const tp = tradePartners[pi];
  const existing = S.tradeDeals.find(deal => !deal.isSell && deal.partnerIdx === pi && deal.resource === res);
  if(existing){ addEvent('ℹ その輸入ルートはすでにあります', ''); return; }
  const deal = {partnerIdx:pi, resource:res, resourceName:r.name, cost:0, amount:0, amountDraft:S.tradeAmountDraft, mode:S.tradeDraftMode, unit:r.unit || '', isSell:false, pct, priceBase:r.price || 10, cooldown:0};
  recalcTradeDeal(deal);
  S.tradeDeals.push(deal);
  addEvent(`🚢 ${tp.name} から ${r.name} の船便契約を開始（${deal.travelWeeks}週）`,'good');
  renderPanel();
};

window.startExport = function(pi, key, pct=S.tradeQuantityDraft){
  const exportable = getExportableResource(key);
  if(!exportable) return;
  const tp = tradePartners[pi];
  const existing = S.tradeDeals.find(deal => deal.isSell && deal.partnerIdx === pi && deal.resource === key);
  if(existing){ addEvent('ℹ その輸出ルートはすでにあります', ''); return; }
  const deal = {partnerIdx:pi, resource:key, resourceName:exportable.name, cost:0, amount:0, amountDraft:S.tradeAmountDraft, mode:S.tradeDraftMode, unit:resources[exportable.resource]?.unit || '', isSell:true, actualResource:exportable.resource, pct, priceBase:exportable.price, cooldown:0};
  recalcTradeDeal(deal);
  S.tradeDeals.push(deal);
  addEvent(`🚢 ${tp.name} へ ${exportable.name} の船便契約を開始（${deal.travelWeeks}週）`,'good');
  renderPanel();
};

window.adjustTradeDeal = function(idx, delta){
  const deal = S.tradeDeals[idx];
  if(!deal) return;
  if((deal.mode || 'percent') === 'amount'){
    const amountStep = Math.max(1, Math.round((deal.amountDraft || deal.amount || 1) * 0.2));
    deal.amountDraft = Math.max(1, Math.round((deal.amountDraft || deal.amount || 1) + delta * amountStep));
  } else {
    const step = deal.pct >= 50 ? 10 : deal.pct >= 25 ? 5 : 1;
    deal.pct = clamp(deal.pct + delta * step, 1, 100);
  }
  recalcTradeDeal(deal);
  renderPanel();
};

window.stopTradeDeal = function(idx){
  if(idx >= 0 && idx < S.tradeDeals.length){
    addEvent(`🚢 貿易停止: ${S.tradeDeals[idx].resourceName}`,'');
    S.tradeDeals.splice(idx, 1);
  }
  renderPanel();
};

// ============================================================
// DEVELOP
// ============================================================
function renderDevelop(){
  let html = '<div class="section"><h3>🏗 地域開発</h3><p style="font-size:12px;color:#8796ad">開発で資源産出増加。高レベルで新素材発見の可能性。ボタンは左クリック長押しで連続実行できます。</p><div class="hold-hint">長押しすると資金と政治資本がある限り連続で開発します。</div>';
  Object.entries(regions).forEach(([k,r])=>{
    const maxed = r.developed >= r.devSlots;
    const baseCost = r.isSea ? 1500 : 600;
    const cost = Math.round(baseCost + r.developed * (r.isSea ? 800 : 400));
    const polCost = r.isSea ? 10 : 5;
    const infraDiscount = 1 - S.infraLevel * 0.03;
    const finalCost = Math.round(cost * Math.max(0.5, infraDiscount));
    html += `<div style="margin:3px 0;padding:5px;background:rgba(0,0,0,.2);border-radius:4px;border-left:3px solid ${r.isSea?'#00BCD4':'#4ecca3'}">
      <div style="font-size:10px;font-weight:bold">${r.isSea?'🌊':'🏔'}${r.name} Lv${r.developed}/${r.devSlots}
        <span class="infotip">?<span class="infotip-text">資源: ${r.resources.map(s=>resources[s]?.name||s).join(', ')}。開発で毎週の産出量増加。Lv5+で新素材発見チャンス(${(getDiscoveryChance(r.developed)*100).toFixed(1)}%)</span></span>
      </div>
      <div style="font-size:8px;color:#888">${r.resources.map(s=>resources[s]?.name||s).join(', ')}</div>
      ${maxed?'<span style="font-size:8px;color:#f39c12">最大開発済</span>':
        `<button class="btn btn-sm btn-green" onmousedown="startRepeatAction('developRegion','${k}')" onmouseup="stopRepeatAction()" onmouseleave="stopRepeatAction()">開発 💰${formatMoneyCompact(finalCost)} 🏛${polCost}</button>`}
    </div>`;
  });
  // Show discovered materials
  if(S.discoveredMaterials.length > 0){
    html += '<div style="margin-top:8px"><h3 style="font-size:11px;color:#f39c12">✦ 発見済み特殊素材</h3>';
    S.discoveredMaterials.forEach(dm=>{
      const mat = [...uniqueMaterials, ...spaceMaterials].find(m=>m.id===dm.id);
      if(!mat) return;
      html += `<div style="padding:4px;margin:2px 0;background:rgba(0,0,0,.2);border-radius:3px;border-left:3px solid ${mat.color}">
        <div style="font-size:10px;font-weight:bold;color:${mat.color}">✦ ${mat.name}</div>
        <div style="font-size:8px;color:#aaa">${mat.desc} (${regions[dm.region]?.name || getSpaceBody(dm.region)?.name || dm.region})</div>
      </div>`;
    });
    html += '</div>';
  }
  html += '</div>';
  return html;
}

function getDiscoveryChance(level){
  if(level < 5) return 0;
  return Math.min(0.15, (level - 4) * 0.012);
}

window.developRegion = function(rk){
  const r = regions[rk];
  if(r.developed >= r.devSlots) return;
  const baseCost = r.isSea ? 1500 : 600;
  const cost = Math.round((baseCost + r.developed * (r.isSea ? 800 : 400)) * Math.max(0.5, 1 - S.infraLevel * 0.03));
  const polCost = r.isSea ? 10 : 5;
  if(S.money < cost){ addEvent('❌ 資金不足','bad'); return; }
  if(S.polCap < polCost){ addEvent('❌ 政治資本不足','bad'); return; }
  S.money -= cost; S.polCap -= polCost;
  r.developed++;
  
  // Resource production increase (smaller per level)
  const prodBoost = Math.max(0.5, 3 - r.developed * 0.05);
  r.resources.forEach(res=>{
    if(resources[res]){
      resources[res].amount += resources[res].max * 0.005 * prodBoost;
      resources[res].max += resources[res].max * 0.02; // Increase max
    }
  });
  
  addEvent(`🏗 ${r.name} 開発完了 (Lv${r.developed})`,'good');
  
  // Discovery chance for unique materials
  const chance = getDiscoveryChance(r.developed);
  if(Math.random() < chance){
    // Find undiscovered material
    const discovered = S.discoveredMaterials.map(d=>d.id);
    const available = uniqueMaterials.filter(m=>!discovered.includes(m.id));
    if(available.length > 0){
      const mat = available[Math.floor(Math.random()*available.length)];
      registerDiscoveredMaterial(mat.id, rk);
      // Create as resource
      resources[mat.id] = {name:mat.name, amount:100, max:10000, color:mat.color, domestic:true, region:rk, price:mat.price, priceHist:[], unit:'kg', isUnique:true};
      for(let i=0;i<50;i++) resources[mat.id].priceHist.push(mat.price*(0.9+Math.random()*0.2));
      addEvent(`✦ 新素材「${mat.name}」を${regions[rk].name}で発見！${mat.desc}`,'good');
      // Also chance for existing known resources
    } else {
      // All unique found, give random existing resource boost
      const rr = r.resources[Math.floor(Math.random()*r.resources.length)];
      if(resources[rr]){
        resources[rr].amount += resources[rr].max * 0.1;
        addEvent(`⛏ ${r.name}で${resources[rr].name}の大規模鉱脈を発見！`,'good');
      }
    }
  } else if(r.developed % 5 === 0 && Math.random() < 0.4){
    // Every 5 levels, chance to discover existing resource not in region
    const allRes = Object.keys(resources).filter(rk2 => !r.resources.includes(rk2) && resources[rk2].domestic !== false);
    if(allRes.length > 0){
      const newRes = allRes[Math.floor(Math.random()*allRes.length)];
      r.resources.push(newRes);
      addEvent(`⛏ ${r.name}で新たに${resources[newRes].name}の産出を確認！`,'good');
    }
  }
  
  // Sea bubble risk
  if(r.isSea){
    S.bubbleIndex += 8;
    if(S.bubbleIndex > 60 && !S.bubbleActive && (S.totalWeeks || 0) >= (S.nextBubbleEligibleWeek || 260)) triggerBubble();
  }
  
  renderPanel(); drawMap();
};

// ============================================================
// BANK
// ============================================================
function renderBank(){
  const debt = getTotalDebt();
  const loanCeiling = getBankLoanCeiling();
  const loanHeadroom = Math.max(0, loanCeiling - debt);
  const loanHeadroomCho = Math.max(1, Math.floor(loanHeadroom / 10000));
  const weeklyInterest = S.loans.reduce((sum, loan) => sum + loan.amount * loan.rate / 52, 0);
  const weeklyPrincipal = S.loans.reduce((sum, loan) => sum + Math.min(loan.amount, loan.weeklyPrincipal || 0), 0);
  let html = `<div class="section"><h3>🏦 日本銀行</h3>
    <div class="macro-grid" style="margin-bottom:6px">
      <div class="macro-card"><div class="macro-label">国債残高</div><div class="macro-value">${formatMoneyCompact(debt)}</div></div>
      <div class="macro-card"><div class="macro-label">借入上限</div><div class="macro-value">${formatMoneyCompact(loanCeiling)}</div><div class="compact-note">残り ${formatMoneyCompact(loanHeadroom)}</div></div>
      <div class="macro-card"><div class="macro-label">週次利払い</div><div class="macro-value">${formatMoneyCompact(weeklyInterest)}</div></div>
      <div class="macro-card"><div class="macro-label">週次元本返済</div><div class="macro-value">${formatMoneyCompact(weeklyPrincipal)}</div></div>
      <div class="macro-card"><div class="macro-label">平均残期間</div><div class="macro-value">${S.loans.length ? Math.round(S.loans.reduce((sum, loan) => sum + (loan.amount / Math.max(1, loan.weeklyPrincipal || 1)), 0) / S.loans.length) : 0}週</div></div>
    </div>
    <div style="padding:6px;background:rgba(0,0,0,.2);border-radius:4px;margin:4px 0">
      <div style="font-size:10px;font-weight:bold;color:#f39c12">公開市場操作
        <span class="infotip">?<span class="infotip-text">引締めは過熱景気・バブル・インフレを抑え、緩和は不況時のGDPと雇用を下支えします。</span></span>
      </div>
      <div style="display:flex;gap:4px;flex-wrap:wrap;margin-top:4px">
        <button class="btn btn-sm btn-yellow" onclick="doOpenMarketOps('tighten')" ${(S.bubbleIndex < 8 && S.businessCycle < 0.16 && S.inflation < 3.6)?'disabled':''}>
          引締め 🏛${26+Math.floor(S.bubbleIndex/3)}</button>
        <button class="btn btn-sm btn-blue" onclick="doOpenMarketOps('ease')" ${(S.businessCycle > -0.12 && S.unemployment < 6.2)?'disabled':''}>
          緩和 🏛22</button>
      </div>
      <div class="compact-note" style="margin-top:4px">景気波: ${(S.businessCycle*100).toFixed(1)} / バブル: ${S.bubbleIndex.toFixed(0)}% / 物価: ${S.inflation.toFixed(1)}%</div>
    </div>
    <div style="padding:6px;background:rgba(0,0,0,.2);border-radius:4px;margin:4px 0">
      <div style="font-size:10px;font-weight:bold;color:#4ecca3">国債発行
        <span class="infotip">?<span class="infotip-text">日銀から資金借入。毎週利息が発生。</span></span>
      </div>
      <div style="display:flex;gap:3px;flex-wrap:wrap;margin-top:3px">
        <button class="btn btn-sm btn-green" onclick="takeLoan(5000,0.015)" ${loanHeadroom < 5000 ? 'disabled' : ''}>${formatMoneyCompact(5000)} (1.5%)</button>
        <button class="btn btn-sm btn-green" onclick="takeLoan(10000,0.02)" ${loanHeadroom < 10000 ? 'disabled' : ''}>${formatMoneyCompact(10000)} (2%)</button>
        <button class="btn btn-sm btn-green" onclick="takeLoan(20000,0.025)" ${loanHeadroom < 20000 ? 'disabled' : ''}>${formatMoneyCompact(20000)} (2.5%)</button>
      </div>
      <div class="save-actions" style="margin-top:8px">
        <input type="number" min="1" max="${loanHeadroomCho}" step="10" value="${Math.min(Math.round(S.bankLoanDraftCho || 1000), loanHeadroomCho)}" onchange="setBankLoanDraftCho(+this.value)" style="width:140px;padding:8px;background:#0f1530;color:#fff;border:1px solid rgba(255,255,255,.08);border-radius:8px">
        <button class="btn btn-sm btn-blue" onclick="takeCustomLoan()" ${loanHeadroom <= 0 ? 'disabled' : ''}>任意発行 (${formatMoneyCompact(Math.round((Math.min(S.bankLoanDraftCho || 1000, loanHeadroomCho)) * 10000))})</button>
      </div>
      <div class="compact-note" style="margin-top:4px">現在の初期債務は ${formatMoneyCompact(10000000)} 相当です。進行度、GDP、国家スコアに応じて借入上限が広がります。</div>
    </div>`;
  if(S.loans.length > 0){
    html += `<div style="padding:6px;background:rgba(0,0,0,.2);border-radius:4px;margin:4px 0">
      <div style="font-size:10px;font-weight:bold;color:#e94560">借入残高</div>`;
    S.loans.forEach((l,i)=>{
      html += `<div class="trade-item"><span>${formatMoneyCompact(l.amount)} (利率 ${(l.rate*100).toFixed(2)}% / 元本 ${formatMoneyCompact(Math.round(l.weeklyPrincipal || 0))}/週)</span>
        <button class="btn btn-sm" onclick="repayLoan(${i})">返済</button></div>`;
    });
    html += '</div>';
  }
  html += '</div>';
  return html;
}

function getBankLoanCeiling(){
  const progressWeeks = S.totalWeeks || 0;
  const score = S.nationalScore || 0;
  const gdp = S.gdp || 0;
  const tech = S.techLevel || 0;
  return Math.round(12000000 + progressWeeks * 25000 + score * 500000 + gdp * 900 + tech * 25000);
}

window.doOpenMarketOps = function(mode='tighten'){
  const polCost = mode === 'tighten' ? 26 + Math.floor(S.bubbleIndex / 3) : 22;
  if(S.polCap < polCost){ addEvent('❌ 政治資本不足','bad'); return; }
  S.polCap -= polCost;
  if(mode === 'tighten'){
    S.gdp -= S.gdp * 0.008;
    S.bubbleIndex = Math.max(0, S.bubbleIndex - 30);
    S.businessCycle -= 0.08;
    S.loans.forEach(loan => { loan.amount *= 0.987; });
    S.bubbleActive = false;
    S.nextBubbleEligibleWeek = Math.max(S.nextBubbleEligibleWeek || 0, (S.totalWeeks || 0) + 260);
    companies.forEach(company => {
      if(company.sector === S.bubbleSector) company.price = Math.round(company.price * rand(0.84, 0.92));
      else company.price = Math.round(company.price * rand(0.96, 0.99));
    });
    addEvent('🏦 日銀が引締めを実施。過熱景気とバブルを抑えた', '');
  } else {
    S.gdp *= 1.006;
    S.money += 2200;
    S.bubbleIndex += 6;
    S.businessCycle += 0.09;
    companies.forEach(company => {
      if(['construction','retail','finance','auto'].includes(company.sector)) company.price = Math.round(company.price * rand(1.01, 1.04));
    });
    addEvent('🏦 日銀が金融緩和を実施。不況下の需要を下支えした', 'good');
  }
  pushHistoryPoint(true);
  renderPanel();
};

window.takeLoan = function(amount, rate){
  if(S.polCap < 5){ addEvent('❌ 政治資本不足','bad'); return; }
  const headroom = Math.max(0, getBankLoanCeiling() - getTotalDebt());
  if(amount > headroom){ addEvent('❌ 現在の進行度ではこれ以上借りられません', 'bad'); return; }
  S.money += amount;
  const termWeeks = 156;
  S.loans.push({amount, rate, original:amount, termWeeks, weeklyPrincipal:amount / termWeeks});
  S.polCap -= 5;
  addEvent(`🏦 ${formatMoneyCompact(amount)} を借入 (${(rate*100).toFixed(2)}% / ${termWeeks}週返済)`,'good');
  pushHistoryPoint(true);
  updateUI(); renderPanel();
};

window.setBankLoanDraftCho = function(value){
  const headroomCho = Math.max(1, Math.floor(Math.max(0, getBankLoanCeiling() - getTotalDebt()) / 10000));
  S.bankLoanDraftCho = clamp(Number(value) || 1000, 1, headroomCho);
  if(S.currentPanel === 'bank') renderPanel();
};

window.takeCustomLoan = function(){
  const maxCho = Math.max(1, Math.floor(Math.max(0, getBankLoanCeiling() - getTotalDebt()) / 10000));
  const amountCho = clamp(Number(S.bankLoanDraftCho) || 1000, 1, maxCho);
  const amount = Math.round(amountCho * 10000);
  const rate = amountCho >= 5000 ? 0.0038 : amountCho >= 2000 ? 0.0032 : 0.0028;
  if(S.polCap < 8){ addEvent('❌ 政治資本不足','bad'); return; }
  if(amount > Math.max(0, getBankLoanCeiling() - getTotalDebt())){ addEvent('❌ 現在の進行度ではこれ以上借りられません', 'bad'); return; }
  S.money += amount;
  const termWeeks = 41600;
  S.loans.push({amount, rate, original:amount, termWeeks, weeklyPrincipal:Math.max(60, amount / termWeeks)});
  S.polCap -= 8;
  addEvent(`🏦 任意国債を ${formatMoneyCompact(amount)} 発行`, 'good');
  pushHistoryPoint(true);
  updateUI();
  renderPanel();
};

window.repayLoan = function(idx){
  const l = S.loans[idx];
  if(S.money < l.amount){ addEvent('❌ 返済資金不足','bad'); return; }
  S.money -= l.amount;
  S.loans.splice(idx, 1);
  addEvent(`🏦 ${formatMoneyCompact(l.amount)} を返済完了`,'good');
  pushHistoryPoint(true);
  updateUI(); renderPanel();
};

// ============================================================
// INFRASTRUCTURE
// ============================================================
function renderInfra(){
  const activeWork = S.draggingWork ? publicWorksCatalog[S.draggingWork.type] : null;
  const activeRoute = S.draggingRoute ? transportNetworkCatalog[S.draggingRoute.type] : null;
  const selectedInfraFilters = getInfraMapFilterList();
  const allInfraSelected = !selectedInfraFilters.length;
  const buildingCount = S.publicWorks.filter(work => work.status === 'building').length;
  const activeCount = S.publicWorks.filter(work => work.status === 'active').length;
  const routeBuildingCount = (S.transportNetworks || []).filter(route => route.status === 'building').length;
  const routeActiveCount = (S.transportNetworks || []).filter(route => route.status === 'active').length;
  const routeGroups = getTransportRouteGroups();
  const prefectureStatList = Object.values(S.prefectureStats || {});
  const avgAppeal = prefectureStatList.length ? prefectureStatList.reduce((sum, stat) => sum + (stat.appeal || 50), 0) / prefectureStatList.length : 50;
  return `<div class="section"><h3>🏗 公共事業・発電</h3>
    <p class="compact-note">カードを押してから日本マップにドラッグして配置します。建設には数か月かかり、完成後は個別に稼働率を調整できます。稼働率が高いほど固定費・燃料費・エネルギー需要も増えます。</p>
    <div class="macro-grid">
      <div class="macro-card"><div class="macro-label">エネルギー供給</div><div class="macro-value">${S.energy.production.toFixed(1)}</div><div class="compact-note">需要 ${S.energy.demand.toFixed(1)} / 不足 ${Math.max(0, S.energy.demand - S.energy.production).toFixed(1)}</div></div>
      <div class="macro-card"><div class="macro-label">建設状況</div><div class="macro-value">${activeCount} 稼働 / ${buildingCount} 建設中</div><div class="compact-note">インフラLv ${S.infraLevel} / 連続配置 ${S.buildRepeatMode ? 'ON' : 'OFF'}</div></div>
    </div>
    ${activeWork ? `<div class="drag-project-pill">配置中: ${activeWork.icon} ${activeWork.name}。日本マップへドラッグして離すと着工します。${S.buildRepeatMode ? '続けて次の一基もそのまま配置できます。' : ''}</div>` : ''}
    <div class="save-actions" style="margin:8px 0">
      <button class="btn btn-sm ${S.buildRepeatMode ? 'active' : ''}" onclick="toggleBuildRepeatMode()">${S.buildRepeatMode ? '連続配置ON' : '連続配置OFF'}</button>
      ${S.draggingWork ? `<button class="btn btn-sm" onclick="cancelWorkPlacement()">配置終了</button>` : ''}
      ${S.draggingStrike ? `<button class="btn btn-sm" onclick="cancelStrikePlacement()">核攻撃モード終了</button>` : ''}
    </div>
    <div class="section" style="margin-top:8px">
      <h3>🗺 地図表示フィルタ</h3>
      <div class="compact-note">公共タブ中にマップへ何を表示するかを選べます。複数選択できます。配置開始中はその種類だけが優先表示されます。</div>
      <div class="save-actions" style="margin-top:8px">
        <button class="btn btn-sm ${allInfraSelected?'active':''}" onclick="setInfraMapFilter('all')">全部</button>
        ${Object.values(publicWorksCatalog).map(work => `<button class="btn btn-sm ${selectedInfraFilters.includes(work.id)?'active':''}" onclick="setInfraMapFilter('${work.id}')">${work.icon}${work.name}</button>`).join('')}
        ${Object.values(transportNetworkCatalog).map(route => `<button class="btn btn-sm ${selectedInfraFilters.includes(route.id)?'active':''}" onclick="setInfraMapFilter('${route.id}')">${route.icon}${route.name}</button>`).join('')}
      </div>
    </div>
    ${Object.values(publicWorksCatalog).map(work => `<div class="research-card">
      <div class="title"><span>${work.icon} ${work.name}</span><span>${formatMoneyCompact(work.cost)}</span></div>
      <div class="meta">${work.desc}<br>工期 ${work.buildWeeks}週 / 週維持費 ${formatMoneyCompact(work.upkeep)} / ${work.output ? `出力 ${work.output}` : `需要 ${work.energyDemand}`}</div>
      <div class="actions"><button class="btn btn-blue" onmousedown="beginPublicWorkPlacement('${work.id}')">配置開始</button></div>
    </div>`).join('')}
    <div class="section" style="margin-top:8px"><h3>🏗 建設・運用一覧</h3>
      ${S.publicWorks.length ? S.publicWorks.map(work => {
        const config = publicWorksCatalog[work.type];
        const status = work.status === 'building'
          ? `<span class="status-badge warn">建設中 ${work.weeksLeft}週</span>`
          : `<span class="status-badge good">稼働 ${work.operatingRate || 0}%</span>`;
        const regionName = regions[work.region]?.name || work.region;
        const upkeep = Math.round((work.upkeep || config.upkeep || 0) * ((work.operatingRate || 0) / 100 || 0));
        const monthlyUsers = Math.round(work.monthlyUsers || work.weeklyUsers || 0);
        const monthlyRevenue = Math.round(work.monthlyRevenue || work.weeklyRevenue || 0);
        const monthlyBalance = Math.round(work.monthlyBalance || work.weeklyBalance || 0);
        return `<div class="work-card">
          <div class="work-row">
            <div>
              <div style="font-size:14px;font-weight:700">${config.icon} ${config.name}</div>
              <div class="work-meta">${regionName} / ${status}${work.seeded ? ' / 初期設備' : ''}<br>${work.status === 'building' ? `完成予定まで ${work.weeksLeft}週` : `週維持費 ${formatMoneyCompact(upkeep)} / エネルギー ${config.output ? `供給 +${(config.output * (work.operatingRate || 0) / 100).toFixed(1)}` : `需要 +${(config.energyDemand * (work.operatingRate || 0) / 100).toFixed(1)}`} / 使用者 ${monthlyUsers.toLocaleString()} / 月収支 <span style="color:${monthlyBalance >= 0 ? '#8ff0bf' : '#ff9aa7'}">${monthlyBalance >= 0 ? '+' : ''}${formatMoneyCompact(monthlyBalance)}</span> / 月収入 ${formatMoneyCompact(monthlyRevenue)}`}</div>
            </div>
            <div class="save-actions">
              ${work.status === 'active' ? [0,25,50,75,100,120].map(rate => `<button class="btn btn-sm ${Math.round(work.operatingRate || 0)===rate?'active':''}" onclick="setPublicWorkRate('${work.id}', ${rate})">${rate}%</button>`).join('') : ''}
              <button class="btn btn-sm" onclick="removePublicWork('${work.id}')">撤去</button>
            </div>
          </div>
        </div>`;
      }).join('') : '<div class="compact-note">まだ公共事業はありません。</div>'}
    </div>
    <div class="section"><h3>🛤 交通・基盤</h3>
      <div class="macro-grid">
        <div class="macro-card"><div class="macro-label">交通網</div><div class="macro-value">${routeActiveCount} 稼働 / ${routeBuildingCount} 建設中</div><div class="compact-note">交通網がつながる県ほど人口移動と企業成長が起きやすくなります。</div></div>
        <div class="macro-card"><div class="macro-label">平均魅力度</div><div class="macro-value">${avgAppeal.toFixed(1)}</div><div class="compact-note">交通と公共がある県から、より魅力的な県へ人口が自然移動します。</div></div>
      </div>
      ${activeRoute ? `<div class="drag-project-pill">路線敷設中: ${activeRoute.icon} ${activeRoute.name}。始点の県をクリックし、次に終点の県をクリックしてください。${S.draggingRoute.fromPrefectureId ? `始点: ${prefectureMap[S.draggingRoute.fromPrefectureId]?.name || ''}` : 'まだ始点未選択です。'}${S.draggingRoute?.type === 'ferry' ? ' / 航路は沿岸県どうしのみ接続できます。' : ''}</div>` : ''}
      <div class="save-actions" style="margin:8px 0">
        <button class="btn btn-sm ${S.transportRepeatMode ? 'active' : ''}" onclick="toggleTransportRepeatMode()">${S.transportRepeatMode ? '路線連続ON' : '路線連続OFF'}</button>
        ${S.draggingRoute ? `<button class="btn btn-sm" onclick="cancelRoutePlacement()">路線配置終了</button>` : ''}
      </div>
      ${Object.values(transportNetworkCatalog).map(route => `<div class="research-card">
        <div class="title"><span>${route.icon} ${route.name}</span><span>${formatMoneyCompact(route.cost)}</span></div>
        <div class="meta">${route.desc}<br>工期 ${route.buildWeeks}週 / 維持費 ${formatMoneyCompact(route.upkeep)} / 生産性 +${route.productivity.toFixed(1)} / 魅力度 +${route.appeal.toFixed(1)}${route.id === 'ferry' ? '<br>条件: 海に面した県どうしのみ。' : ''}</div>
        <div class="actions"><button class="btn btn-green" onclick="beginTransportPlacement('${route.id}')">路線敷設開始</button></div>
      </div>`).join('')}
      <div class="section" style="margin-top:8px"><h3>🚉 路線一覧</h3>
        ${routeGroups.length ? routeGroups.map(group => {
          const config = transportNetworkCatalog[group.type];
          const status = group.status === 'building' ? `建設中 ${group.weeksLeft}週` : `稼働 ${group.operatingRate}%`;
          return `<div class="work-card">
            <div class="work-row">
              <div>
                <div style="font-size:14px;font-weight:700">${config?.icon || '🛤'} ${config?.name || group.type}</div>
                <div class="work-meta">${group.from?.name || '-'} → ${group.to?.name || '-'} / ${status} / ${group.segmentCount}区間<br>影響: 生産性 +${config?.productivity?.toFixed(1) || '0'} / 魅力度 +${config?.appeal?.toFixed(1) || '0'} / 月利用者 ${Math.round(group.monthlyUsers).toLocaleString()} / 月収支 <span style="color:${group.monthlyBalance >= 0 ? '#8ff0bf' : '#ff9aa7'}">${group.monthlyBalance >= 0 ? '+' : ''}${formatMoneyCompact(group.monthlyBalance)}</span></div>
              </div>
              <div class="save-actions">
                ${group.status === 'active' ? [0,25,50,75,100,120].map(rate => `<button class="btn btn-sm ${Math.round(group.operatingRate || 0)===rate?'active':''}" onclick="setTransportChainRate('${group.chainId}', ${rate})">${rate}%</button>`).join('') : ''}
                <button class="btn btn-sm" onclick="removeTransportChain('${group.chainId}')">撤去</button>
              </div>
            </div>
          </div>`;
        }).join('') : '<div class="compact-note">まだ交通網はありません。</div>'}
      </div>
      <div class="goal-card">
        <div class="goal-title"><span>交通インフラ拡張</span><span>Lv ${S.infraLevel}</span></div>
        <div class="goal-desc">開発コスト減、物流改善、建設企業の受注増に寄与します。</div>
        <button class="btn btn-blue" onclick="upgradeInfra()">強化 💰${3000+S.infraLevel*1500}億 🏛10</button>
      </div>
    </div>
  </div>`;
}

window.beginPublicWorkPlacement = function(workId){
  const config = publicWorksCatalog[workId];
  if(!config) return;
  if(S.money < config.cost){ addEvent('❌ 資金不足', 'bad'); return; }
  if(S.polCap < config.pol){ addEvent('❌ 政治資本不足', 'bad'); return; }
  S.infraMapFilter = workId;
  S.draggingWork = {type:workId, x:regions.kanto.cx, y:regions.kanto.cy};
  S.currentMapMode = 'japan';
  loadViewForMode('japan');
  updateMapHud();
  renderPanel();
  drawMap();
};

window.setInfraMapFilter = function(filter){
  const isValid = filter === 'all' || !!publicWorksCatalog[filter] || !!transportNetworkCatalog[filter];
  if(!isValid) return;
  if(filter === 'all'){
    S.infraMapFilter = 'all';
  } else {
    const current = getInfraMapFilterList();
    const next = current.includes(filter)
      ? current.filter(entry => entry !== filter)
      : current.concat(filter);
    S.infraMapFilter = next.length ? next : 'all';
  }
  renderPanel();
  drawMap();
};

function getInfraMapFilterList(){
  const raw = S.infraMapFilter;
  if(Array.isArray(raw)){
    return raw.filter(id => publicWorksCatalog[id] || transportNetworkCatalog[id]);
  }
  if(typeof raw === 'string' && raw !== 'all' && (publicWorksCatalog[raw] || transportNetworkCatalog[raw])){
    return [raw];
  }
  return [];
}

window.toggleBuildRepeatMode = function(){
  S.buildRepeatMode = !S.buildRepeatMode;
  renderPanel();
};

window.cancelWorkPlacement = function(){
  S.draggingWork = null;
  updateMapHud();
  drawMap();
  renderPanel();
};

window.beginTransportPlacement = function(type){
  const config = transportNetworkCatalog[type];
  if(!config) return;
  if(S.money < config.cost){ addEvent('❌ 資金不足', 'bad'); return; }
  if(S.polCap < config.pol){ addEvent('❌ 政治資本不足', 'bad'); return; }
  S.infraMapFilter = type;
  const anchor = getPrefectureTransitAnchor(S.japanFocusPrefecture) || prefectureMap[S.japanFocusPrefecture] || {x:0.67, y:0.47};
  S.draggingRoute = {
    type,
    fromPrefectureId:'',
    x:anchor.x,
    y:anchor.y,
    chainId:`chain-${Date.now()}-${Math.random().toString(36).slice(2,6)}`,
    segmentIndex:0,
  };
  S.currentMapMode = 'japan';
  loadViewForMode('japan');
  updateMapHud();
  renderPanel();
  drawMap();
};

window.toggleTransportRepeatMode = function(){
  S.transportRepeatMode = !S.transportRepeatMode;
  renderPanel();
};

window.cancelRoutePlacement = function(){
  S.draggingRoute = null;
  updateMapHud();
  drawMap();
  renderPanel();
};

window.setTransportChainRate = function(chainId, rate){
  const routes = (S.transportNetworks || []).filter(route => getTransportRouteChainId(route) === chainId && route.status === 'active');
  if(!routes.length) return;
  routes.forEach(route => {
    route.operatingRate = clamp(rate, 0, 120);
  });
  updateUI();
  renderPanel();
};

window.removeTransportChain = function(chainId){
  const routes = (S.transportNetworks || []).filter(route => getTransportRouteChainId(route) === chainId);
  if(!routes.length) return;
  let salvage = 0;
  routes.forEach(route => {
    const config = transportNetworkCatalog[route.type];
    salvage += Math.round((config?.cost || 0) * (route.status === 'building' ? 0.3 : 0.18));
  });
  S.transportNetworks = S.transportNetworks.filter(route => getTransportRouteChainId(route) !== chainId);
  if(salvage > 0) S.money += salvage;
  addEvent(`🧱 路線群を撤去${salvage ? `。資材回収 ${formatMoneyCompact(salvage)}` : ''}`, '');
  updateUI();
  renderPanel();
  drawMap();
};

window.setPublicWorkRate = function(workId, rate){
  const work = S.publicWorks.find(entry => entry.id === workId);
  if(!work || work.status !== 'active') return;
  work.operatingRate = clamp(rate, 0, 120);
  updateUI();
  renderPanel();
};

window.removePublicWork = function(workId){
  const index = S.publicWorks.findIndex(entry => entry.id === workId);
  if(index < 0) return;
  const work = S.publicWorks[index];
  const config = publicWorksCatalog[work.type];
  const salvage = work.seeded ? 0 : Math.round((config?.cost || 0) * (work.status === 'building' ? 0.35 : 0.22));
  S.publicWorks.splice(index, 1);
  if(salvage > 0) S.money += salvage;
  addEvent(`🧱 ${config?.name || work.type} を撤去${salvage > 0 ? `。資材回収 ${formatMoneyCompact(salvage)}` : ''}`, '');
  updateUI();
  renderPanel();
  drawMap();
};

function placeTransportRoute(fromPrefectureId, toPrefectureId, type){
  if(fromPrefectureId === toPrefectureId){
    addEvent('❌ 同じ県どうしは接続できません', 'bad');
    return false;
  }
  const config = transportNetworkCatalog[type];
  const from = prefectureMap[fromPrefectureId];
  const to = prefectureMap[toPrefectureId];
  if(!config || !from || !to) return false;
  const routeVerdict = validateTransportRoutePlacement(fromPrefectureId, toPrefectureId, type);
  if(!routeVerdict.ok){
    addEvent(`❌ ${routeVerdict.reason}`, 'bad');
    return false;
  }
  if((S.transportNetworks || []).some(route => {
    if(route.type !== type) return false;
    return (route.fromPrefectureId === fromPrefectureId && route.toPrefectureId === toPrefectureId) ||
      (route.fromPrefectureId === toPrefectureId && route.toPrefectureId === fromPrefectureId);
  })){
    addEvent('❌ その路線はすでに存在します', 'bad');
    return false;
  }
  const distance = worldDistance(from.x, from.y, to.x, to.y);
  const distanceFactor = clamp(distance / 0.12, 0.7, 2.3);
  const finalCost = Math.round(config.cost * distanceFactor);
  const polCost = Math.round(config.pol * clamp(0.8 + distanceFactor * 0.35, 0.8, 2));
  if(S.money < finalCost){ addEvent('❌ 路線敷設資金が不足しています', 'bad'); return false; }
  if(S.polCap < polCost){ addEvent('❌ 政治資本不足', 'bad'); return false; }
  S.money -= finalCost;
  S.polCap -= polCost;
  const draggingChainId = S.draggingRoute?.chainId || `chain-${Date.now()}-${Math.random().toString(36).slice(2,6)}`;
  const segmentIndex = typeof S.draggingRoute?.segmentIndex === 'number' ? S.draggingRoute.segmentIndex : ((S.transportNetworks || []).filter(route => getTransportRouteChainId(route) === draggingChainId).length);
  S.transportNetworks.push({
    id:`tn-${Date.now()}-${Math.random().toString(36).slice(2,6)}`,
    chainId:draggingChainId,
    chainLabel:`${from.name} → ${to.name}`,
    segmentIndex,
    type,
    fromPrefectureId,
    toPrefectureId,
    status:'building',
    totalWeeks:Math.max(4, Math.round(config.buildWeeks * distanceFactor)),
    weeksLeft:Math.max(4, Math.round(config.buildWeeks * distanceFactor)),
    operatingRate:82,
    weeklyUsers:0,
    weeklyRevenue:0,
    weeklyBalance:0,
    monthlyUsers:0,
    monthlyRevenue:0,
    monthlyBalance:0,
    monthlyWeeks:0,
  });
  companies.filter(company => ['construction','shipping','auto','telecom'].includes(company.sector)).forEach(company => {
    company.revenue *= rand(1.01, 1.04);
    company.favorability = Math.min(100, company.favorability + 0.9);
  });
  addEvent(`🛤 ${from.name} と ${to.name} を結ぶ ${config.name} に着工`, 'good');
  renderPanel();
  drawMap();
  updateUI();
  return true;
}

window.removeTransportRoute = function(routeId){
  const index = (S.transportNetworks || []).findIndex(route => route.id === routeId);
  if(index < 0) return;
  const route = S.transportNetworks[index];
  const config = transportNetworkCatalog[route.type];
  const salvage = Math.round((config?.cost || 0) * (route.status === 'building' ? 0.3 : 0.18));
  S.transportNetworks.splice(index, 1);
  if(salvage > 0) S.money += salvage;
  addEvent(`🧱 ${config?.name || '交通網'} を撤去${salvage ? `。資材回収 ${formatMoneyCompact(salvage)}` : ''}`, '');
  updateUI();
  renderPanel();
  drawMap();
};

function placePublicWork(workId, worldPos){
  const config = publicWorksCatalog[workId];
  const verdict = canPlacePublicWork(workId, worldPos);
  if(!config || !verdict.ok){
    addEvent(`❌ 配置不可: ${verdict.reason}`, 'bad');
    renderPanel();
    drawMap();
    return;
  }
  S.money -= config.cost;
  S.polCap -= config.pol;
  S.publicWorks.push({
    id:`pw-${Date.now()}-${Math.random().toString(36).slice(2,6)}`,
    type:workId,
    x:worldPos.x,
    y:worldPos.y,
    region:verdict.regionKey,
    workers:100,
    status:'building',
    totalWeeks:config.buildWeeks,
    weeksLeft:config.buildWeeks,
    operatingRate:100,
    upkeep:config.upkeep || 0,
    weeklyUsers:0,
    weeklyRevenue:0,
    weeklyBalance:0,
    monthlyUsers:0,
    monthlyRevenue:0,
    monthlyBalance:0,
    monthlyWeeks:0,
  });
  companies.filter(company => company.sector === 'construction').forEach(company => {
    company.price = Math.round(company.price * rand(1.02,1.06));
    company.revenue *= rand(1.02,1.05);
    company.favorability = Math.min(100, company.favorability + 1.2);
  });
  if(workId === 'industrial') companies.filter(company => ['construction','auto','tech','mining'].includes(company.sector)).forEach(company => company.price = Math.round(company.price * rand(1.015,1.05)));
  addEvent(`🏗 ${regions[verdict.regionKey].name} で ${config.name} に着工。完成まで ${config.buildWeeks}週`, 'good');
  renderPanel();
  drawMap();
  updateUI();
  return true;
}

window.upgradeInfra = function(){
  const cost = 3000 + S.infraLevel * 1500;
  if(S.money < cost){ addEvent('❌ 資金不足','bad'); return; }
  if(S.polCap < 10){ addEvent('❌ 政治資本不足','bad'); return; }
  S.money -= cost; S.polCap -= 10;
  S.infraLevel++;
  companies.filter(c=>c.sector==='construction').forEach(c=>c.price=Math.round(c.price*1.05));
  addEvent(`🔧 インフラLv${S.infraLevel}に強化 開発コスト-${S.infraLevel*3}%`,'good');
  updateUI(); renderPanel();
};

// ============================================================
// CYBER
// ============================================================
function renderCyber(){
  let html = `<div class="section"><h3>💻 サイバー戦力 (${Math.round(S.cyberPower)})</h3>
    <p style="font-size:8px;color:#666">IT企業への投資でサイバー力向上。相手国を上回ると貿易で有利(割引/割増)。</p>
    <div style="padding:6px;background:rgba(0,0,0,.2);border-radius:4px;margin:4px 0">
      <div style="font-size:10px;font-weight:bold">🔒 サイバー防衛強化
        <span class="infotip">?<span class="infotip-text">各国の基準値を引き上げています。以前より高額ですが、強化しないと外貨・貿易・戦争で後手になりやすいです。</span></span>
      </div>
      <button class="btn btn-sm btn-purple" onclick="investCyber(12000)">投資${formatMoneyCompact(12000)} (+4〜6)</button>
      <button class="btn btn-sm btn-purple" onclick="investCyber(28000)">投資${formatMoneyCompact(28000)} (+9〜13)</button>
      <button class="btn btn-sm btn-purple" onclick="investCyber(52000)">大規模投資${formatMoneyCompact(52000)} (+17〜24)</button>
    </div>
    <div style="margin-top:8px">
      <div style="font-size:10px;font-weight:bold;margin-bottom:4px">各国サイバー力比較</div>`;
  tradePartners.forEach(tp=>{
    const advantage = S.cyberPower > tp.cyberPower;
    html += `<div class="trade-item">
      <span>${tp.name}</span>
      <span style="color:${advantage?'#4ecca3':'#e94560'}">${tp.cyberPower} ${advantage?'◀ 優勢':'▶ 劣勢'}</span>
    </div>`;
  });
  html += `</div><div class="section" style="margin-top:8px"><h3>🕶 攻撃オペレーション</h3>`;
  tradePartners.forEach((tp, i) => {
    const chance = clamp(0.22 + (S.cyberPower - tp.cyberPower) * 0.008 + S.techLevel * 0.006, 0.08, 0.86);
    html += `<div class="trade-item"><span>${tp.name}</span><span><button class="btn btn-sm" onclick="launchCyberAttack(${i})">攻撃 ${(chance*100).toFixed(0)}%</button></span></div>`;
  });
  html += '</div></div>';
  return html;
}

window.investCyber = function(amount){
  if(S.money < amount){ addEvent('❌ 資金不足','bad'); return; }
  S.money -= amount;
  const gain = amount <= 12000 ? rand(4, 6) : amount <= 28000 ? rand(9, 13) : rand(17, 24);
  S.cyberPower += gain;
  S.techLevel += gain * 0.16;
  companies.filter(c=>c.sector==='tech').forEach(c=>{
    c.price=Math.round(c.price*rand(1.02,1.05));
    c.revenue *= rand(1.01, 1.03);
    c.favorability = Math.min(100, (c.favorability || 50) + 0.6);
  });
  addEvent(`💻 サイバー力+${gain.toFixed(1)} (現在:${Math.round(S.cyberPower)})`,'good');
  updateUI(); renderPanel();
};

window.launchCyberAttack = function(targetIdx){
  const target = tradePartners[targetIdx];
  const chance = clamp(0.22 + (S.cyberPower - target.cyberPower) * 0.008 + S.techLevel * 0.006, 0.08, 0.86);
  const polCost = 12;
  if(S.polCap < polCost){ addEvent('❌ 政治資本不足', 'bad'); return; }
  S.polCap -= polCost;
  if(Math.random() < chance){
    const steal = Math.round(400 + target.economy * rand(12, 28));
    S.money += steal;
    S.cyberSuccesses = (S.cyberSuccesses || 0) + 1;
    target.relation = Math.max(0, target.relation - 8);
    target.stability = Math.max(0, target.stability - 5);
    if(Math.random() < 0.4) target.cyberPower = Math.max(0, target.cyberPower - 2);
    companies.filter(c=>c.sector==='tech' || c.sector==='defense').forEach(c=>c.price=Math.round(c.price*rand(1.008,1.03)));
  addEvent(`🕶 ${target.name}へのサイバー攻撃成功。${formatMoneyCompact(steal)}を奪取`, 'good');
  } else {
    S.approval = Math.max(0, S.approval - 1.5);
    target.relation = Math.max(0, target.relation - 3);
    addEvent(`🛑 ${target.name}へのサイバー攻撃は失敗。逆探知で支持率低下`, 'bad');
  }
  updateUI();
  renderPanel();
};

function getChaosCatalog(){
  const catalog = {
    meteor:{id:'meteor', category:'japan-harm', name:'隕石投下', icon:'☄', target:'japan', cost:52000, pol:40, radius:0.055, weeks:104, desc:'着弾地点の会社と公共事業を広く破壊し、長期復旧を強います。'},
    blackout:{id:'blackout', category:'japan-harm', name:'大停電工作', icon:'⚡', target:'japan', cost:29500, pol:26, radius:0.046, weeks:58, desc:'発電・通信・医療に大きな悪影響。周辺設備の稼働率も深く落ち込みます。'},
    riot:{id:'riot', category:'japan-harm', name:'暴動扇動', icon:'🪧', target:'japan', cost:22800, pol:23, radius:0.04, weeks:42, desc:'社会不安を局所的に爆発させ、金融・小売・建設へ強い打撃を与えます。'},
    typhoon:{id:'typhoon', category:'japan-harm', name:'超大型台風工作', icon:'🌀', target:'japan', cost:36000, pol:31, radius:0.055, weeks:72, desc:'港湾・電力・食品・物流に大きな損害。沿岸設備も長期間低下します。'},
    wildfire:{id:'wildfire', category:'japan-harm', name:'山火事誘発', icon:'🔥', target:'japan', cost:24600, pol:22, radius:0.042, weeks:54, desc:'電力網や木材供給、周辺都市機能を長く傷つけます。'},
    industrial:{id:'industrial', category:'japan-harm', name:'化学プラント事故', icon:'🏭', target:'japan', cost:30200, pol:26, radius:0.03, weeks:48, desc:'工場地帯・病院・通信網に連鎖障害を起こし、企業収益を大きく削ります。'},
    blockade:{id:'blockade', category:'world-harm', name:'港湾封鎖工作', icon:'🚫', target:'world', cost:35200, pol:32, desc:'対象国の物流と経済を弱らせ、輸送便を遅延・停止させます。'},
    sanctions:{id:'sanctions', category:'world-harm', name:'制裁網工作', icon:'📉', target:'world', cost:39200, pol:34, desc:'対象国の友好・通貨・企業評価を多方面から削り、景気後退を誘発します。'},
    currency:{id:'currency', category:'world-harm', name:'通貨攻撃工作', icon:'💱', target:'world', cost:32800, pol:29, desc:'対象国の通貨と資本流入を弱らせ、外需・市場信認を長く損ねます。'},
  };
  if(S.bioWeaponUnlocks?.includes('amber_fever')){
    catalog.amber_release = {
      id:'amber_release',
      category:'research-threat',
      name:'アンバー熱散布',
      icon:'☣',
      target:'world',
      cost:9800,
      pol:10,
      desc:'武野薬品工業が作った高感染性病原体を世界へ放ちます。',
      pathogenId:'amber_fever',
      sourceCompany:'武野薬品工業'
    };
  }
  if(S.bioWeaponUnlocks?.includes('velvet_strain')){
    catalog.velvet_release = {
      id:'velvet_release',
      category:'research-threat',
      name:'ベルベット株散布',
      icon:'🧬',
      target:'world',
      cost:15400,
      pol:16,
      desc:'武野薬品工業が作った人工変異株で世界経済へ大打撃を与えます。',
      pathogenId:'velvet_strain',
      sourceCompany:'武野薬品工業'
    };
  }
  if(hasCompletedResearch('relief_biologics') || hasCompletedResearch('antiviral_platform')){
    catalog.medical_relief = {
      id:'medical_relief',
      category:'research-benefit',
      name:'医療救援パッケージ',
      icon:'💉',
      target:'japan',
      cost:14800,
      pol:18,
      radius:0.05,
      weeks:24,
      desc:'武野薬品工業の救援物資を投入し、災害地の病院・企業・支持率を回復させます。',
      sourceCompany:'武野薬品工業'
    };
  }
  if(hasCompletedResearch('climate_shield') || hasCompletedResearch('fusion_grid')){
    catalog.grid_restore = {
      id:'grid_restore',
      category:'research-benefit',
      name:'送電網緊急復旧',
      icon:'🔋',
      target:'japan',
      cost:16800,
      pol:20,
      radius:0.052,
      weeks:26,
      desc:'東都電力HD の緊急送電支援で、電力系公共施設と沿線企業を立て直します。',
      sourceCompany:'東都電力HD'
    };
  }
  if(hasCompletedResearch('urban_resilience') || hasCompletedResearch('quantum_network')){
    catalog.digital_shield = {
      id:'digital_shield',
      category:'research-benefit',
      name:'都市防災シールド',
      icon:'🛰',
      target:'japan',
      cost:15900,
      pol:18,
      radius:0.048,
      weeks:22,
      desc:'NTテレコムの防災デジタルツインを使い、災害地域の生産性と安定度を回復します。',
      sourceCompany:'NTテレコム'
    };
  }
  return catalog;
}

function getNearbyCompanies(worldPos, radius){
  return companies.filter(company => worldDistance(company.mapX, company.mapY, worldPos.x, worldPos.y) <= radius);
}

function getNearbyWorks(worldPos, radius){
  return S.publicWorks.filter(work => worldDistance(work.x, work.y, worldPos.x, worldPos.y) <= radius);
}

function spendChaosCost(config){
  if(S.money < config.cost){ addEvent('❌ 資金不足', 'bad'); return false; }
  if(S.polCap < config.pol){ addEvent('❌ 政治資本不足', 'bad'); return false; }
  S.money -= config.cost;
  S.polCap -= config.pol;
  S.weeklyBreakdown.operations = (S.weeklyBreakdown.operations || 0) + config.cost;
  return true;
}

function getSelectedChaosCompany(){
  const company = getCompanyByName(S.selectedChaosCompany) || companies[0];
  if(company) S.selectedChaosCompany = company.name;
  return company;
}

window.setChaosCompanyTarget = function(name){
  S.selectedChaosCompany = name;
  renderPanel();
};

window.triggerCompanyShock = function(companyName, action='scandal'){
  const company = getCompanyByName(companyName);
  if(!company) return;
  const actionConfig = {
    scandal:{cost:3800, pol:9, icon:'🕵', label:'不祥事工作', price:[0.66,0.86], revenue:[0.78,0.93], favor:[7,16], sentiment:[-0.22,-0.08], techHit:[2,7]},
    sabotage:{cost:5200, pol:12, icon:'💣', label:'サボタージュ', price:[0.54,0.78], revenue:[0.68,0.88], favor:[10,20], sentiment:[-0.3,-0.12], techHit:[5,10]},
    pump:{cost:4300, pol:10, icon:'📣', label:'買い煽り', price:[1.05,1.15], revenue:[1.01,1.05], favor:[1,4], sentiment:[0.06,0.16], techHit:[0,0]},
  }[action];
  if(!actionConfig){ addEvent('❌ 工作内容が不明です', 'bad'); return; }
  if(S.money < actionConfig.cost || S.polCap < actionConfig.pol){ addEvent('❌ 実行資源不足', 'bad'); return; }
  S.money -= actionConfig.cost;
  S.polCap -= actionConfig.pol;
  S.weeklyBreakdown.operations = (S.weeklyBreakdown.operations || 0) + actionConfig.cost;
  const isPositive = action === 'pump';
  company.price = Math.max(getCompanyDynamicFloor(company), Math.round(company.price * rand(actionConfig.price[0], actionConfig.price[1])));
  company.base = Math.max(getCompanyDynamicFloor(company), Math.round(company.base * (isPositive ? rand(1.0, 1.03) : rand(0.9, 0.99))));
  company.revenue = Math.max(120, Math.round(company.revenue * rand(actionConfig.revenue[0], actionConfig.revenue[1])));
  const favorDelta = rand(actionConfig.favor[0], actionConfig.favor[1]) * (isPositive ? 1 : -1);
  company.favorability = clamp((company.favorability || 50) + favorDelta, 0, 100);
  company.sentiment = clamp((company.sentiment || 0) + rand(actionConfig.sentiment[0], actionConfig.sentiment[1]), -0.9, 0.9);
  if(!isPositive){
    company.techProgress = Math.max(0, (company.techProgress || 0) - rand(actionConfig.techHit[0], actionConfig.techHit[1]));
    S.socialUnrest = clamp(S.socialUnrest + rand(1.5, 5), 0, 100);
  }
  addEvent(`${actionConfig.icon} ${company.name} へ ${actionConfig.label} を実施`, isPositive ? 'good' : 'bad');
  renderPanel();
  updateUI();
};

function applyChaosDamageToCompanies(targetCompanies, options={}){
  const sectorAllow = options.sectors || null;
  targetCompanies.forEach(company => {
    if(sectorAllow && !sectorAllow.includes(company.sector)) return;
    company.price = Math.max(getCompanyDynamicFloor(company), Math.round(company.price * rand(options.priceMin || 0.7, options.priceMax || 0.92)));
    company.base = Math.max(getCompanyDynamicFloor(company), Math.round(company.base * rand(options.baseMin || 0.94, options.baseMax || 0.995)));
    company.revenue = Math.max(120, Math.round(company.revenue * rand(options.revMin || 0.8, options.revMax || 0.96)));
    company.favorability = clamp((company.favorability || 50) - rand(options.favorMin || 3, options.favorMax || 8), 0, 100);
    company.sentiment = clamp((company.sentiment || 0) - rand(0.05, 0.18), -0.9, 0.9);
  });
}

function applyChaosDamageToWorks(targetWorks, options={}){
  targetWorks.forEach(work => {
    if(options.types && !options.types.includes(work.type)) return;
    if(Math.random() < (options.destroyChance || 0)){
      S.publicWorks = S.publicWorks.filter(entry => entry.id !== work.id);
      return;
    }
    work.damage = Math.max(work.damage || 0, rand(options.damageMin || 0.45, options.damageMax || 0.9));
    work.operatingRate = Math.max(0, (work.operatingRate || 100) - rand(options.rateMin || 18, options.rateMax || 50));
  });
}

function renderChaos(){
  const chaosCatalog = getChaosCatalog();
  const selectedCompany = getSelectedChaosCompany();
  const groupedConfigs = [
    {id:'japan-harm', title:'🇯🇵 国内災害', body:'日本マップの地点指定で大きな損害を与える工作です。'},
    {id:'world-harm', title:'🌐 対外工作', body:'世界マップの対象国を選んで長期のダメージを与えます。'},
    {id:'research-threat', title:'☣ 研究由来の脅威', body:'研究で解禁した会社由来の災害兵器です。誰が作ったかも表示されます。'},
    {id:'research-benefit', title:'🛡 研究由来の恩恵', body:'研究で解禁した善性イベントです。被災地の復旧や安定化に使えます。'},
  ];
  return `<div class="section"><h3>🌋 人為イベント / 災害工作</h3>
    <div class="compact-note">狙って相場を揺らすための高コスト行動です。日本マップ指定型は実際の位置に近い企業と公共事業へ被害を与え、世界型は対象国の物流と景気を直撃します。</div>
    ${S.draggingDisaster ? `<div class="drag-project-pill">配置中: ${chaosCatalog[S.draggingDisaster.type]?.icon || '⚠'} ${chaosCatalog[S.draggingDisaster.type]?.name}. ${chaosCatalog[S.draggingDisaster.type]?.target === 'world' ? '世界マップ' : '日本マップ'}をクリックして発動。</div>` : ''}
    <div class="save-actions" style="margin-top:8px">
      ${S.draggingDisaster ? `<button class="btn btn-sm" onclick="cancelChaosPlacement()">配置終了</button>` : ''}
      <button class="btn btn-sm ${S.chaosRepeatMode ? 'active' : ''}" onclick="toggleChaosRepeatMode()">${S.chaosRepeatMode ? '連続発動ON' : '連続発動OFF'}</button>
    </div>
    ${groupedConfigs.map(group => {
      const items = Object.values(chaosCatalog).filter(config => config.category === group.id);
      if(!items.length) return '';
      return `<div class="section" style="margin-top:8px"><h3>${group.title}</h3><div class="compact-note">${group.body}</div>
        ${items.map(config => `<div class="research-card ${config.category==='research-benefit' ? 'good' : ''}">
          <div class="title"><span>${config.icon} ${config.name}</span><span>${formatMoneyCompact(config.cost || 0)} / 🏛${config.pol || 0}</span></div>
          <div class="meta">${config.desc}<br>${config.target === 'world' ? '世界マップ対象国クリック型' : `${config.category==='research-benefit' ? '日本マップ支援地点指定型' : '日本マップ地点指定型'}${config.weeks ? ` / 効果残存 ${config.weeks}週` : ''}`}${config.sourceCompany ? `<br><span style="color:${config.category==='research-benefit' ? '#8ff0bf' : '#ffb3c0'}">提供会社: ${config.sourceCompany}</span>` : ''}</div>
          <div class="actions"><button class="btn btn-yellow" onclick="beginChaosPlacement('${config.id}')">発動準備</button></div>
        </div>`).join('')}
      </div>`;
    }).join('')}
    <div class="section" style="margin-top:8px"><h3>📉 セクター工作</h3>
      <div class="compact-note">不祥事リークや買い煽りで、カテゴリ単位に人為的な急変を起こします。</div>
      ${Object.entries(sectors).map(([sectorId, sector]) => `<div class="trade-item"><span>${sector.icon} ${sector.name}</span><span><button class="btn btn-sm" onclick="triggerSectorShock('${sectorId}','down')">不祥事</button> <button class="btn btn-sm btn-green" onclick="triggerSectorShock('${sectorId}','up')">買い煽り</button></span></div>`).join('')}
    </div>
    <div class="section" style="margin-top:8px"><h3>🏢 個別企業工作</h3>
      <div class="compact-note">個々の会社だけを狙って不祥事、サボタージュ、買い煽りを実行できます。セクター全体工作より狙い撃ちです。</div>
      <select style="width:100%;padding:8px;background:#0f1530;color:#fff;border:1px solid rgba(255,255,255,.08);border-radius:6px" onchange="setChaosCompanyTarget(this.value)">
        ${companies.map(company => `<option value="${company.name}" ${selectedCompany?.name===company.name?'selected':''}>${company.name} / ${sectors[company.sector]?.name || company.sector}</option>`).join('')}
      </select>
      ${selectedCompany ? `<div class="support-row">
        <div style="font-size:12px;font-weight:700">${selectedCompany.name}</div>
        <div class="compact-note">株価 ${formatMoneyCompact(selectedCompany.price)} / 売上 ${formatMoneyCompact(Math.round(selectedCompany.revenue))} / 好感度 ${Math.round(selectedCompany.favorability || 50)}</div>
        <div class="save-actions" style="margin-top:8px">
          <button class="btn btn-sm" onclick="triggerCompanyShock('${selectedCompany.name}','scandal')">🕵 不祥事</button>
          <button class="btn btn-sm" onclick="triggerCompanyShock('${selectedCompany.name}','sabotage')">💣 サボタージュ</button>
          <button class="btn btn-sm btn-green" onclick="triggerCompanyShock('${selectedCompany.name}','pump')">📣 買い煽り</button>
        </div>
      </div>` : ''}
    </div>
  </div>`;
}

window.toggleChaosRepeatMode = function(){
  S.chaosRepeatMode = !S.chaosRepeatMode;
  renderPanel();
};

window.beginChaosPlacement = function(type){
  const config = getChaosCatalog()[type];
  if(!config) return;
  const focusCountry = getWorldCountry(S.worldFocusCountry);
  S.draggingDisaster = {
    type,
    target:config.target,
    x:config.target === 'world' ? focusCountry.cx : 0.58,
    y:config.target === 'world' ? focusCountry.cy : 0.48
  };
  storeCurrentView();
  S.currentMapMode = config.target === 'world' ? 'world' : 'japan';
  loadViewForMode(S.currentMapMode);
  updateMapHud();
  renderPanel();
  drawMap();
  addEvent(`🌋 ${config.name} の対象位置を選択してください`, '');
};

window.cancelChaosPlacement = function(){
  S.draggingDisaster = null;
  updateMapHud();
  renderPanel();
  drawMap();
};

window.triggerSectorShock = function(sectorId, direction='down'){
  const sector = sectors[sectorId];
  if(!sector) return;
  const cost = direction === 'down' ? 4500 : 5200;
  const pol = direction === 'down' ? 10 : 12;
  if(S.money < cost || S.polCap < pol){ addEvent('❌ 実行資源不足', 'bad'); return; }
  S.money -= cost;
  S.polCap -= pol;
  const isUp = direction === 'up';
  companies.filter(company => company.sector === sectorId).forEach(company => {
    const priceMult = isUp ? rand(1.03, 1.1) : rand(0.68, 0.88);
    const revenueMult = isUp ? rand(1.01, 1.05) : rand(0.8, 0.95);
    company.price = Math.max(getCompanyDynamicFloor(company), Math.round(company.price * priceMult));
    company.revenue = Math.max(120, Math.round(company.revenue * revenueMult));
    company.sentiment = clamp((company.sentiment || 0) + (isUp ? rand(0.05, 0.14) : -rand(0.08, 0.2)), -0.9, 0.9);
  });
  if(!isUp) S.socialUnrest = clamp(S.socialUnrest + rand(2, 6), 0, 100);
  addEvent(`${isUp ? '📣' : '🕵'} ${sector.name} セクターへ ${isUp ? '買い煽り' : '不祥事工作'} を実施`, isUp ? 'good' : 'bad');
  renderPanel();
  updateUI();
};

function applyChaosAtPoint(type, worldPos){
  const config = getChaosCatalog()[type];
  if(!config) return false;
  const regionKey = getClosestRegion(worldPos.x, worldPos.y);
  if(!regionKey || regions[regionKey]?.isSea){ addEvent('❌ 日本列島の陸地を選んでください', 'bad'); return false; }
  if(!spendChaosCost(config)) return false;
  const nearbyCompanies = getNearbyCompanies(worldPos, config.radius);
  const nearbyWorks = getNearbyWorks(worldPos, config.radius);
  if(type === 'meteor'){
    spawnMapEffect({mode:'japan', x:worldPos.x, y:worldPos.y, radius:config.radius, life:28, label:'隕石衝突', icon:'☄', color:[255,120,92]});
    regions[regionKey].disaster = {type:'隕石衝突', severity:0.94, weeksLeft:config.weeks, totalWeeks:config.weeks, x:worldPos.x, y:worldPos.y, radius:config.radius, startedAt:Date.now()};
    applyChaosDamageToCompanies(nearbyCompanies, {priceMin:0.14, priceMax:0.42, baseMin:0.46, baseMax:0.72, revMin:0.28, revMax:0.62, favorMin:8, favorMax:18});
    applyChaosDamageToWorks(nearbyWorks, {destroyChance:0.62, damageMin:0.82, damageMax:1, rateMin:42, rateMax:78});
    companies.filter(company => ['construction','mining'].includes(company.sector)).forEach(company => company.price = Math.round(company.price * rand(1.02, 1.08)));
    S.socialUnrest = clamp(S.socialUnrest + 14, 0, 100);
    S.stability = Math.max(0, S.stability - 8);
    S.gdp *= 0.991;
    addEvent(`☄ ${regions[regionKey].name} に隕石が落下。周辺企業と公共事業が甚大な被害`, 'bad');
  } else if(type === 'blackout'){
    spawnMapEffect({mode:'japan', x:worldPos.x, y:worldPos.y, radius:config.radius, life:22, label:'大停電', icon:'⚡', color:[255,220,110]});
    regions[regionKey].disaster = {type:'大停電', severity:0.7, weeksLeft:config.weeks, totalWeeks:config.weeks, x:worldPos.x, y:worldPos.y, radius:config.radius, startedAt:Date.now()};
    applyChaosDamageToWorks(nearbyWorks, {types:['thermal','hydro','solar','hospital','industrial'], destroyChance:0.12, damageMin:0.62, damageMax:0.92, rateMin:28, rateMax:58});
    applyChaosDamageToCompanies(nearbyCompanies, {sectors:['energy','telecom','tech','hospital','retail','defense'], priceMin:0.64, priceMax:0.86, baseMin:0.9, baseMax:0.98, revMin:0.72, revMax:0.92, favorMin:4, favorMax:10});
    S.energyBonus = Math.max(0, S.energyBonus - 3.5);
    S.socialUnrest = clamp(S.socialUnrest + 6, 0, 100);
    S.stability = Math.max(0, S.stability - 3);
    S.gdp *= 0.996;
    addEvent(`⚡ ${regions[regionKey].name} で大停電。発電・通信・医療が混乱`, 'bad');
  } else if(type === 'riot'){
    spawnMapEffect({mode:'japan', x:worldPos.x, y:worldPos.y, radius:config.radius, life:18, label:'暴動', icon:'🪧', color:[255,132,132]});
    regions[regionKey].disaster = {type:'暴動', severity:0.58, weeksLeft:config.weeks, totalWeeks:config.weeks, x:worldPos.x, y:worldPos.y, radius:config.radius, startedAt:Date.now()};
    applyChaosDamageToCompanies(nearbyCompanies, {sectors:['finance','retail','construction','shipping','food'], priceMin:0.66, priceMax:0.88, baseMin:0.9, baseMax:0.985, revMin:0.76, revMax:0.93, favorMin:5, favorMax:11});
    S.approval = Math.max(0, S.approval - 4.8);
    S.socialUnrest = clamp(S.socialUnrest + 10, 0, 100);
    S.stability = Math.max(0, S.stability - 6);
    S.gdp *= 0.996;
    addEvent(`🪧 ${regions[regionKey].name} で暴動が発生。金融と小売が大きく下落`, 'bad');
  } else if(type === 'typhoon'){
    spawnMapEffect({mode:'japan', x:worldPos.x, y:worldPos.y, radius:config.radius, life:26, label:'超大型台風', icon:'🌀', color:[122,187,255]});
    regions[regionKey].disaster = {type:'超大型台風', severity:0.76, weeksLeft:config.weeks, totalWeeks:config.weeks, x:worldPos.x, y:worldPos.y, radius:config.radius, startedAt:Date.now()};
    applyChaosDamageToCompanies(nearbyCompanies, {sectors:['shipping','food','retail','energy','construction','hospital'], priceMin:0.58, priceMax:0.84, baseMin:0.88, baseMax:0.98, revMin:0.72, revMax:0.9, favorMin:4, favorMax:10});
    applyChaosDamageToWorks(nearbyWorks, {destroyChance:0.24, damageMin:0.55, damageMax:0.88, rateMin:24, rateMax:54});
    ['fish','rice','timber'].forEach(resourceKey => { if(resources[resourceKey]) resources[resourceKey].amount *= rand(0.78, 0.9); });
    S.energyBonus = Math.max(0, S.energyBonus - 2);
    S.socialUnrest = clamp(S.socialUnrest + 8, 0, 100);
    S.approval = Math.max(0, S.approval - 2.8);
    S.stability = Math.max(0, S.stability - 5);
    S.gdp *= 0.993;
    addEvent(`🌀 ${regions[regionKey].name} を超大型台風が直撃。港湾・食品・電力が深刻な損害`, 'bad');
  } else if(type === 'wildfire'){
    spawnMapEffect({mode:'japan', x:worldPos.x, y:worldPos.y, radius:config.radius, life:20, label:'山火事', icon:'🔥', color:[255,116,72]});
    regions[regionKey].disaster = {type:'山火事', severity:0.62, weeksLeft:config.weeks, totalWeeks:config.weeks, x:worldPos.x, y:worldPos.y, radius:config.radius, startedAt:Date.now()};
    applyChaosDamageToCompanies(nearbyCompanies, {sectors:['construction','food','hospital','retail','energy'], priceMin:0.64, priceMax:0.86, baseMin:0.9, baseMax:0.98, revMin:0.74, revMax:0.92, favorMin:4, favorMax:9});
    applyChaosDamageToWorks(nearbyWorks, {destroyChance:0.18, damageMin:0.48, damageMax:0.82, rateMin:20, rateMax:44});
    if(resources.timber) resources.timber.amount *= rand(0.76, 0.88);
    S.socialUnrest = clamp(S.socialUnrest + 5, 0, 100);
    S.stability = Math.max(0, S.stability - 4);
    S.gdp *= 0.996;
    addEvent(`🔥 ${regions[regionKey].name} 周辺で大規模山火事。物流と都市機能が麻痺`, 'bad');
  } else if(type === 'industrial'){
    spawnMapEffect({mode:'japan', x:worldPos.x, y:worldPos.y, radius:config.radius, life:18, label:'化学プラント事故', icon:'🏭', color:[255,184,90]});
    regions[regionKey].disaster = {type:'化学プラント事故', severity:0.68, weeksLeft:config.weeks, totalWeeks:config.weeks, x:worldPos.x, y:worldPos.y, radius:config.radius, startedAt:Date.now()};
    applyChaosDamageToCompanies(nearbyCompanies, {sectors:['construction','energy','hospital','tech','telecom','pharma'], priceMin:0.52, priceMax:0.82, baseMin:0.84, baseMax:0.97, revMin:0.66, revMax:0.88, favorMin:6, favorMax:14});
    applyChaosDamageToWorks(nearbyWorks, {destroyChance:0.22, damageMin:0.58, damageMax:0.9, rateMin:28, rateMax:58});
    S.socialUnrest = clamp(S.socialUnrest + 6, 0, 100);
    S.approval = Math.max(0, S.approval - 2.2);
    S.stability = Math.max(0, S.stability - 4.5);
    S.gdp *= 0.995;
    addEvent(`🏭 ${regions[regionKey].name} で化学プラント事故。工業地帯と医療網に連鎖障害`, 'bad');
  } else if(type === 'medical_relief'){
    spawnMapEffect({mode:'japan', x:worldPos.x, y:worldPos.y, radius:config.radius, life:20, label:'医療救援パッケージ', icon:'💉', color:[124,240,186]});
    if(regions[regionKey].disaster){
      regions[regionKey].disaster.severity = Math.max(0.12, regions[regionKey].disaster.severity * 0.7);
      regions[regionKey].disaster.weeksLeft = Math.max(10, Math.round(regions[regionKey].disaster.weeksLeft * 0.78));
    }
    nearbyWorks.forEach(work => {
      if(work.type === 'hospital') work.operatingRate = Math.min(120, (work.operatingRate || 100) + 22);
      work.damage = Math.max(0, (work.damage || 0) - 0.28);
    });
    nearbyCompanies.forEach(company => {
      if(['hospital','pharma','food','retail'].includes(company.sector)){
        company.revenue *= rand(1.05, 1.14);
        company.price = Math.round(company.price * rand(1.02, 1.08));
        company.favorability = Math.min(100, (company.favorability || 50) + 4);
      }
    });
    S.approval = Math.min(100, S.approval + 2.8);
    S.stability = Math.min(100, S.stability + 2.2);
    addEvent(`💉 武野薬品工業の医療救援パッケージを ${regions[regionKey].name} へ投入。病院と住民生活が改善`, 'good');
  } else if(type === 'grid_restore'){
    spawnMapEffect({mode:'japan', x:worldPos.x, y:worldPos.y, radius:config.radius, life:20, label:'送電網緊急復旧', icon:'🔋', color:[120,214,255]});
    nearbyWorks.forEach(work => {
      if(['thermal','hydro','solar','hospital','industrial'].includes(work.type)){
        work.damage = Math.max(0, (work.damage || 0) - 0.34);
        work.operatingRate = Math.min(120, (work.operatingRate || 100) + 18);
      }
    });
    nearbyCompanies.forEach(company => {
      if(['energy','telecom','tech','construction'].includes(company.sector)){
        company.revenue *= rand(1.04, 1.11);
        company.price = Math.round(company.price * rand(1.02, 1.07));
        company.favorability = Math.min(100, (company.favorability || 50) + 3);
      }
    });
    S.energyBonus += 1.1;
    addEvent(`🔋 東都電力HD の送電網緊急復旧を ${regions[regionKey].name} へ投入。電力と通信が安定`, 'good');
  } else if(type === 'digital_shield'){
    spawnMapEffect({mode:'japan', x:worldPos.x, y:worldPos.y, radius:config.radius, life:20, label:'都市防災シールド', icon:'🛰', color:[156,188,255]});
    if(regions[regionKey].disaster){
      regions[regionKey].disaster.severity = Math.max(0.1, regions[regionKey].disaster.severity * 0.76);
    }
    nearbyCompanies.forEach(company => {
      company.revenue *= rand(1.03, 1.09);
      company.price = Math.round(company.price * rand(1.015, 1.06));
      company.favorability = Math.min(100, (company.favorability || 50) + 2.5);
    });
    const regionPrefectures = prefectures.filter(prefecture => prefecture.region === regionKey);
    regionPrefectures.forEach(prefecture => {
      const stat = S.prefectureStats?.[prefecture.id];
      if(!stat) return;
      stat.productivity = clamp(stat.productivity + 3.5, -35, 90);
      stat.appeal = clamp((stat.appeal || 50) + 2.6, 0, 100);
    });
    S.stability = Math.min(100, S.stability + 1.8);
    addEvent(`🛰 NTテレコムの都市防災シールドを ${regions[regionKey].name} へ展開。地域の生産性と復旧速度が向上`, 'good');
  }
  if(!S.chaosRepeatMode) S.draggingDisaster = null;
  renderPanel();
  updateUI();
  return true;
}

function applyChaosToCountry(type, countryId){
  const config = getChaosCatalog()[type];
  const country = getWorldCountry(countryId);
  const partner = getWorldCountryPartner(country);
  if(!config || !partner || country.id === 'japan'){ addEvent('❌ 対象国を選択してください', 'bad'); return false; }
  if(config.pathogenId){
    if(S.activeMajorEvent){ addEvent('❌ すでに重大イベントが進行中です', 'bad'); return false; }
    const pathogen = pathogenCatalog.find(entry => entry.id === config.pathogenId);
    if(!pathogen){ addEvent('❌ 病原体データが見つかりません', 'bad'); return false; }
    if(!spendChaosCost(config)) return false;
    triggerBioOutbreak(pathogen);
    if(!S.chaosRepeatMode) S.draggingDisaster = null;
    renderPanel();
    updateUI();
    return true;
  }
  if(!spendChaosCost(config)) return false;
  if(type === 'blockade'){
    spawnMapEffect({mode:'world', x:country.cx, y:country.cy, radius:0.07, life:22, label:'港湾封鎖', icon:'🚫', color:[233,108,108]});
    partner.economy = Math.max(6, partner.economy - rand(8, 18));
    partner.stability = Math.max(0, partner.stability - rand(7, 14));
    partner.relation = Math.max(0, partner.relation - rand(5, 11));
    partner.hostility = Math.min(100, (partner.hostility || 0) + rand(8, 18));
    partner.gdpUsd = Math.max(0.08, partner.gdpUsd * rand(0.86, 0.95));
    partner.gdp = scalePartnerGdp(partner.gdpUsd);
    partner.nuclearScars = (partner.nuclearScars || 0) + rand(3, 7);
    S.tradeShipments = S.tradeShipments.filter(shipment => shipment.partnerIdx !== country.partnerIdx);
    S.tradeDeals.filter(deal => deal.partnerIdx === country.partnerIdx).forEach(deal => {
      deal.cooldown = Math.max(deal.cooldown || 0, 12);
    });
    S.foreignCompanies.filter(company => company.countryId === country.id).forEach(company => {
      company.price = Math.max(100, Math.round(company.price * rand(0.68, 0.9)));
      company.revenue = Math.max(120, Math.round(company.revenue * rand(0.72, 0.92)));
    });
    addEvent(`🚫 ${country.name} への港湾封鎖工作。物流と企業収益に打撃`, 'bad');
  } else if(type === 'sanctions'){
    spawnMapEffect({mode:'world', x:country.cx, y:country.cy, radius:0.075, life:24, label:'制裁網', icon:'📉', color:[255,188,92]});
    partner.relation = Math.max(0, partner.relation - rand(8, 16));
    partner.stability = Math.max(0, partner.stability - rand(6, 13));
    partner.hostility = Math.min(100, (partner.hostility || 0) + rand(7, 16));
    partner.economy = Math.max(5, partner.economy - rand(5, 12));
    partner.gdpUsd = Math.max(0.08, partner.gdpUsd * rand(0.84, 0.94));
    partner.currencyRate = clamp(partner.currencyRate * rand(0.88, 0.97), partner.baseCurrencyRate * 0.6, partner.baseCurrencyRate * 1.35);
    partner.gdp = scalePartnerGdp(partner.gdpUsd);
    S.foreignCompanies.filter(company => company.countryId === country.id).forEach(company => {
      company.price = Math.max(100, Math.round(company.price * rand(0.62, 0.88)));
      company.revenue = Math.max(120, Math.round(company.revenue * rand(0.74, 0.91)));
      company.favorability = clamp((company.favorability || 50) - rand(3, 8), 0, 100);
    });
    addEvent(`📉 ${country.name} に制裁網工作。通貨・友好・企業評価が同時に悪化`, 'bad');
  } else if(type === 'currency'){
    spawnMapEffect({mode:'world', x:country.cx, y:country.cy, radius:0.072, life:20, label:'通貨攻撃', icon:'💱', color:[120,210,255]});
    partner.currencyRate = clamp(partner.currencyRate * rand(0.82, 0.94), partner.baseCurrencyRate * 0.55, partner.baseCurrencyRate * 1.35);
    partner.gdpUsd = Math.max(0.08, partner.gdpUsd * rand(0.88, 0.96));
    partner.economy = Math.max(5, partner.economy - rand(3, 9));
    partner.stability = Math.max(0, partner.stability - rand(4, 10));
    partner.relation = Math.max(0, partner.relation - rand(3, 7));
    partner.gdp = scalePartnerGdp(partner.gdpUsd);
    S.foreignCompanies.filter(company => company.countryId === country.id).forEach(company => {
      company.price = Math.max(100, Math.round(company.price * rand(0.72, 0.9)));
      company.revenue = Math.max(120, Math.round(company.revenue * rand(0.8, 0.95)));
    });
    addEvent(`💱 ${country.name} へ通貨攻撃工作。為替と株式市場が大きく動揺`, 'bad');
  }
  if(!S.chaosRepeatMode) S.draggingDisaster = null;
  renderPanel();
  updateUI();
  return true;
}

// ============================================================
// NUCLEAR
// ============================================================
function renderNuke(){
  const militaryCatalog = getMilitaryCatalog();
  const militaryPower = getMilitaryPowerContribution();
  const plantWeeks = Math.max(12, Math.ceil(18 - S.techLevel * 0.15));
  const missileWeeks = Math.max(18, Math.ceil(30 - S.techLevel * 0.1));
  const plantChance = clamp(0.46 + S.techLevel * 0.018 + (resources.uranium?.amount || 0) / 10000 * 0.14, 0.2, 0.9);
  const missileChance = clamp(0.22 + S.techLevel * 0.012 + S.defPower * 0.003, 0.08, 0.72);
  let html = `<div class="section"><h3>🪖 戦力総覧</h3>
    <p class="compact-note">歩兵・戦車・航空・艦艇などの通常戦力に加え、諜報班で敵国へ潜入して情報を奪取できます。軍事企業・造船・自動車・IT・通信の連携が強いほど量産と品質が伸びます。</p>
    <div class="macro-grid">
      <div class="macro-card"><div class="macro-label">通常戦力</div><div class="macro-value">${Math.round(militaryPower)}</div><div class="compact-note">歩兵 ${S.militaryInventory.infantry} / 戦車 ${S.militaryInventory.tanks} / 航空 ${S.militaryInventory.fighters}</div></div>
      <div class="macro-card"><div class="macro-label">諜報班</div><div class="macro-value">${S.militaryInventory.spies || 0}</div><div class="compact-note">潜入中 ${(S.spyMissions || []).length} / 情報奪取や敵戦力把握に使用</div></div>
      <div class="macro-card"><div class="macro-label">原発 / 核戦力</div><div class="macro-value">${S.nuclearPlants} / ${S.nukePower}</div><div class="compact-note">抑止ドクトリン ${S.nuclear.doctrine.toFixed(0)}</div></div>
    </div>
    <div class="section" style="margin-top:8px"><h3>🪖 通常戦力生産</h3>
      <div class="compact-note">会社の技術方針と好感度が、そのまま生産週数と品質に効きます。軍事企業・自動車・造船・ITの連携が強いほど短く強くなります。</div>
      ${Object.values(militaryCatalog).map(unit => {
        const boost = getMilitaryCompanyBoost(unit.id);
        const visual = getMilitaryVisualProfile(unit.id, boost);
        const qualityMultiplier = getMilitaryQualityMultiplier(unit.id, boost);
        const weeks = Math.max(6, Math.round(unit.weeks * Math.max(0.62, 1 - boost * 0.1)));
        const cost = Math.round(unit.cost * Math.max(0.68, 1 - boost * 0.1));
        const suppliers = getMilitarySuppliers(unit.id).slice(0, 3).map(company => company.name).join(' / ') || '協力企業なし';
        const tierPills = visual
          ? `<div class="save-actions" style="margin-top:2px">${visual.labels.map((label, index) => `<span class="pill" style="${index === visual.tier ? 'background:rgba(255,214,102,.18);color:#ffe7a6;border:1px solid rgba(255,214,102,.24)' : ''}">${index === visual.tier ? '★ ' : ''}${label}</span>`).join('')}</div>`
          : '';
        if(visual){
          return `<div class="goal-card">
            <div class="military-art-card">
              <div class="military-art-visual">
                <div class="art" style="background-image:url('${visual.image}')"></div>
                <div class="badge">${visual.label}</div>
              </div>
              <div class="military-art-details">
                <div class="goal-title"><span>${unit.icon} ${unit.name}</span><span>品質 ${qualityMultiplier.toFixed(2)}x</span></div>
      <div class="goal-desc">${unit.note}<br>所要 ${weeks}週 / コスト 💰${formatMoneyCompact(cost)} / 🏛${unit.pol} / 防衛力 +${(unit.power * (1 + boost * 0.42)).toFixed(1)}<br>協力: ${suppliers}</div>
                <div class="military-art-sub">現在の装備世代: <strong style="color:#f6fbff">${visual.label}</strong> / 技術成熟 ${visual.maturity}% / 次段階 ${visual.nextLabel}</div>
                ${tierPills}
                <div class="save-actions"><button class="btn btn-blue" onclick="buildMilitaryUnit('${unit.id}',1)">x1</button><button class="btn btn-sm" onclick="buildMilitaryUnit('${unit.id}',5)">x5</button><button class="btn btn-sm" onclick="buildMilitaryUnit('${unit.id}',10)">x10</button><button class="btn btn-sm" onclick="buildMilitaryUnit('${unit.id}',25)">x25</button><button class="btn btn-sm" onclick="buildMilitaryUnit('${unit.id}',100)">x100</button></div>
              </div>
            </div>
          </div>`;
        }
        return `<div class="goal-card">
          <div class="goal-title"><span>${unit.icon} ${unit.name}</span><span>品質 ${qualityMultiplier.toFixed(2)}x</span></div>
      <div class="goal-desc">${unit.note}<br>所要 ${weeks}週 / コスト 💰${formatMoneyCompact(cost)} / 🏛${unit.pol} / 防衛力 +${(unit.power * (1 + boost * 0.42)).toFixed(1)}<br>協力: ${suppliers}</div>
          <div class="save-actions"><button class="btn btn-blue" onclick="buildMilitaryUnit('${unit.id}',1)">x1</button><button class="btn btn-sm" onclick="buildMilitaryUnit('${unit.id}',5)">x5</button><button class="btn btn-sm" onclick="buildMilitaryUnit('${unit.id}',10)">x10</button><button class="btn btn-sm" onclick="buildMilitaryUnit('${unit.id}',25)">x25</button><button class="btn btn-sm" onclick="buildMilitaryUnit('${unit.id}',100)">x100</button></div>
        </div>`;
      }).join('')}
      <div class="macro-grid" style="margin-top:8px">
        <div class="macro-card"><div class="macro-label">歩兵旅団</div><div class="macro-value">${S.militaryInventory.infantry}</div></div>
        <div class="macro-card"><div class="macro-label">戦車大隊</div><div class="macro-value">${S.militaryInventory.tanks}</div></div>
        <div class="macro-card"><div class="macro-label">戦闘機隊</div><div class="macro-value">${S.militaryInventory.fighters}</div></div>
        <div class="macro-card"><div class="macro-label">艦艇隊 / 無人機群</div><div class="macro-value">${S.militaryInventory.ships} / ${S.militaryInventory.drones}</div></div>
        <div class="macro-card"><div class="macro-label">諜報班</div><div class="macro-value">${S.militaryInventory.spies || 0}</div></div>
      </div>
    </div>
    <div class="section" style="margin-top:8px"><h3>☢ 核・原子力</h3>
      <div class="compact-note">原子力は長期建設、核抑止は超高難度計画です。核攻撃は相手国の経済・人口・企業・物流に長い傷跡を残しますが、国際関係と国内支持率も大きく削ります。</div>
      <div class="goal-card" style="margin-top:8px">
        <div class="goal-title"><span>⚡ 原子力発電所建設</span><span>${(plantChance*100).toFixed(0)}%</span></div>
        <div class="goal-desc">所要 ${plantWeeks}週 | コスト 💰10,000億 / 🏛28 | 成功時: 原発+1 / エネルギー+5% / GDP基盤強化</div>
        <button class="btn btn-yellow" onclick="buildNukePlant()" ${S.nuclear.projects.length>=2?'disabled':''}>建設開始</button>
      </div>
      <div class="goal-card">
        <div class="goal-title"><span>🚀 核ミサイル開発</span><span>${(missileChance*100).toFixed(0)}%</span></div>
        <div class="goal-desc">所要 ${missileWeeks}週 | コスト 💰24,000億 / 🏛75 | 成功時: 核+1 / 防衛力+24 / 各国友好悪化</div>
        <button class="btn" style="border-color:#e94560" onclick="buildNuke()" ${S.nuclear.projects.length>=2?'disabled':''}>研究開始</button>
      </div>
      <div class="goal-card">
        <div class="goal-title"><span>🎯 核ミサイル発射</span><span>${S.nukePower} 発</span></div>
        <div class="goal-desc">世界マップで対象国をクリックして発射します。発射モードは連続選択にも対応します。</div>
        <div class="save-actions">
          <button class="btn btn-yellow" onclick="beginNuclearStrikePlacement()" ${S.nukePower <= 0 ? 'disabled' : ''}>発射モード</button>
          <button class="btn btn-sm ${S.strikeRepeatMode ? 'active' : ''}" onclick="toggleStrikeRepeatMode()">${S.strikeRepeatMode ? '連続発射ON' : '連続発射OFF'}</button>
        </div>
      </div>
    </div>`;
  if(S.militaryProjects.length){
    html += `<div class="section" style="margin-top:8px"><h3>⚙ 生産キュー</h3>
      ${S.militaryProjects.map(project => {
        const unit = militaryCatalog[project.type];
        const pct = Math.round((1 - project.weeksLeft / project.totalWeeks) * 100);
        return `<div class="goal-card">
          <div class="goal-title"><span>${unit?.icon || '🪖'} ${unit?.name || project.type}</span><span>${pct}%</span></div>
          <div class="goal-desc">残り ${project.weeksLeft}週 / 数量 x${project.qty || 1} / 配備時 防衛力 +${project.powerGain.toFixed(1)} / 協力 ${project.suppliers.join(' / ') || 'なし'}</div>
          <div class="goal-progress"><div class="fill" style="width:${pct}%"></div></div>
        </div>`;
      }).join('')}
    </div>`;
  }
  if(S.nuclear.projects.length){
    html += `<div class="section" style="margin-top:8px"><h3>🧪 進行中プロジェクト</h3>`;
    S.nuclear.projects.forEach(project => {
      const pct = Math.round((1 - project.weeksLeft / project.totalWeeks) * 100);
      html += `<div class="goal-card">
        <div class="goal-title"><span>${project.type === 'plant' ? '原発建設' : '核開発'}</span><span>${pct}%</span></div>
        <div class="goal-desc">${project.weeksLeft}週 残り</div>
        <div class="goal-progress"><div class="fill" style="width:${pct}%"></div></div>
      </div>`;
    });
    html += `</div>`;
  }
  html += `<div style="margin-top:6px;font-size:10px">
      <div>各国核戦力比較:</div>
      ${tradePartners.map(tp=>`<div class="trade-item"><span>${tp.name}</span><span>${tp.nukePower}</span></div>`).join('')}
    </div>
  </div>`;
  return html;
}

window.buildNukePlant = function(){
  const weeks = Math.max(12, Math.ceil(18 - S.techLevel * 0.15));
  if(S.money < 10000){ addEvent('❌ 資金不足','bad'); return; }
  if(S.polCap < 28){ addEvent('❌ 政治資本不足','bad'); return; }
  S.money -= 10000; S.polCap -= 28;
  S.nuclear.projects.push({type:'plant', totalWeeks:weeks, weeksLeft:weeks});
  addEvent(`☢ 原子力発電所の建設を開始。完成まで ${weeks}週`, 'good');
  updateUI(); renderPanel();
};

window.buildNuke = function(){
  const weeks = Math.max(18, Math.ceil(30 - S.techLevel * 0.1));
  if(S.money < 24000){ addEvent('❌ 資金不足','bad'); return; }
  if(S.polCap < 75){ addEvent('❌ 政治資本不足','bad'); return; }
  S.money -= 24000; S.polCap -= 75;
  S.nuclear.projects.push({type:'missile', totalWeeks:weeks, weeksLeft:weeks});
  addEvent(`🚀 核抑止研究を開始。完成まで ${weeks}週`, 'bad');
  updateUI(); renderPanel();
};

window.beginNuclearStrikePlacement = function(){
  if(S.nukePower <= 0){ addEvent('❌ 使用可能な核ミサイルがありません', 'bad'); return; }
  S.draggingStrike = {type:'nuke'};
  S.currentMapMode = 'world';
  loadViewForMode('world');
  updateMapHud();
  drawMap();
  addEvent('☢ 世界マップで対象国をクリックして核攻撃します', 'bad');
};

window.cancelStrikePlacement = function(){
  S.draggingStrike = null;
  drawMap();
  renderPanel();
};

window.toggleStrikeRepeatMode = function(){
  S.strikeRepeatMode = !S.strikeRepeatMode;
  renderPanel();
};

function fireNuclearStrike(countryId){
  const country = getWorldCountry(countryId);
  const partner = getWorldCountryPartner(country);
  if(!partner || country.id === 'japan') return;
  if(S.nukePower <= 0){ addEvent('❌ 核ミサイル残数がありません', 'bad'); return; }
  S.nukePower--;
  partner.relation = 0;
  partner.stability = Math.max(0, partner.stability - 34);
  partner.economy = Math.max(4, partner.economy - 28);
  partner.cyberPower = Math.max(16, partner.cyberPower - 12);
  partner.population = Math.max(1, partner.population - rand(4, 34));
  partner.gdpUsd = Math.max(0.08, partner.gdpUsd * rand(0.72, 0.88));
  partner.gdp = scalePartnerGdp(partner.gdpUsd);
  partner.nuclearScars = (partner.nuclearScars || 0) + rand(18, 34);
  S.foreignCompanies.filter(company => company.countryId === countryId).forEach(company => {
    company.price = Math.max(100, Math.round(company.price * rand(0.1, 0.36)));
    company.base = Math.max(100, Math.round(company.base * rand(0.52, 0.78)));
    company.revenue = Math.max(120, Math.round(company.revenue * rand(0.24, 0.58)));
  });
  S.tradeShipments = S.tradeShipments.filter(shipment => shipment.partnerIdx !== country.partnerIdx);
  S.tradeDeals = S.tradeDeals.filter(deal => deal.partnerIdx !== country.partnerIdx);
  if(S.activeWar?.countryId === countryId){
    subtractUnits(S.activeWar.enemyFront, {
      infantry:Math.round((S.activeWar.enemyFront.infantry || 0) * rand(0.45, 0.8)),
      tanks:Math.round((S.activeWar.enemyFront.tanks || 0) * rand(0.35, 0.75)),
      fighters:Math.round((S.activeWar.enemyFront.fighters || 0) * rand(0.28, 0.7)),
      ships:Math.round((S.activeWar.enemyFront.ships || 0) * rand(0.24, 0.68)),
      drones:Math.round((S.activeWar.enemyFront.drones || 0) * rand(0.42, 0.85)),
    });
    getUnitKeys().forEach(key => {
      S.activeWar.enemyReserves[key] = Math.max(0, Math.round((S.activeWar.enemyReserves[key] || 0) * rand(0.58, 0.86)));
    });
    S.activeWar.enemyDispatches = [];
    pushWarLog(`${country.name} 本土へ核攻撃。敵軍の輸送と予備戦力に壊滅的打撃`, 'bad');
    partner.hostility = Math.min(100, (partner.hostility || 0) + 22);
  }
  tradePartners.forEach(other => { if(other !== partner) other.relation = Math.max(0, other.relation - 8); });
  S.approval = Math.max(0, S.approval - 9);
  S.stability = Math.max(0, S.stability - 8);
  addEvent(`☢ ${country.name} に核攻撃。経済・人口・企業・輸送網へ壊滅的打撃`, 'bad');
  if(!S.strikeRepeatMode || S.nukePower <= 0) S.draggingStrike = null;
  updateUI();
  renderPanel();
}

// ============================================================
// BOTTOM BAR ACTIONS
// ============================================================
function openAction(type){
  const modal = document.getElementById('actionModal');
  const title = document.getElementById('modalTitle');
  const content = document.getElementById('modalContent');
  const modalCard = modal.querySelector('.modal');
  if(modalCard) modalCard.classList.remove('modal-world-country','modal-cyber','modal-finance');
  if(S.draggingWork) S.draggingWork = null;
  const t = currentI18n();
  modal.dataset.action = type;
  disposeModalDom();
  
  switch(type){
    case 'budget':
      title.textContent = S.language === 'en' ? '💰 Budget Allocation' : '💰 予算配分';
      content.innerHTML = renderPolicy();
      break;
    case 'subsidy':
      title.textContent = '🏭 補助金';
      content.innerHTML = `<p style="font-size:10px;color:#aaa;margin-bottom:6px">特定セクターに補助金投入。予算配分より大きな即時効果。バブルリスクあり。</p>
        ${Object.entries(sectors).map(([k,s])=>`
          <button class="btn" style="margin:2px;border-color:${s.color}" onclick="giveSubsidy('${k}')">
            ${s.icon} ${s.name} (💰3,000億 🏛8)
            <span class="infotip">?<span class="infotip-text">該当セクターの需給改善、投資家心理改善、関連セクターへの波及が起こります。特殊素材があると技術革新が起こりやすくなります。</span></span>
          </button>`).join('')}`;
      break;
    case 'diplomacy':
      title.textContent = '🌐 外交';
      content.innerHTML = `<p style="font-size:10px;color:#aaa;margin-bottom:6px">外交関係改善で貿易コスト削減。</p>
        ${tradePartners.map((tp,i)=>`
          <div style="display:flex;justify-content:space-between;align-items:center;padding:4px;margin:2px 0;background:rgba(0,0,0,.2);border-radius:3px">
            <span style="font-size:10px">${tp.name} (友好:${tp.relation}) ${S.cyberPower>tp.cyberPower?'💻優勢':''}</span>
            <button class="btn btn-sm btn-blue" onclick="improveDiplomacy(${i})">強化 🏛8</button>
          </div>`).join('')}`;
      break;
    case 'military':
      title.textContent = '⚔ 軍事';
      content.innerHTML = `<p style="font-size:10px;color:#aaa;margin-bottom:6px">防衛力強化。関連企業株上昇。</p>
        <div style="font-size:10px;margin-bottom:6px">現在の防衛力: <strong>${S.defPower}</strong></div>
        <button class="btn" onclick="militaryAction('equip')">🛡装備更新 💰${formatMoneyCompact(3000)} 🏛10 <span class="infotip">?<span class="infotip-text">防衛装備の即応性が上がり、防衛企業に追い風が入ります。</span></span></button>
        <button class="btn" onclick="militaryAction('base')">🏢基地拡張 💰${formatMoneyCompact(5000)} 🏛15 <span class="infotip">?<span class="infotip-text">防衛力だけでなく、建設や周辺産業の受注にも効きます。</span></span></button>
        <button class="btn" onclick="militaryAction('rd')">🔬防衛R&D 💰${formatMoneyCompact(4000)} 🏛8 <span class="infotip">?<span class="infotip-text">防衛・技術・サイバーの複合強化につながります。</span></span></button>`;
      break;
    case 'education':
      title.textContent = '📚 教育';
      content.innerHTML = `<button class="btn" onclick="eduAction('uni')">🎓大学投資 💰${formatMoneyCompact(2000)} <span class="infotip">?<span class="infotip-text">長期の技術人材と研究力に寄与します。</span></span></button>
        <button class="btn" onclick="eduAction('stem')">🔬STEM推進 💰${formatMoneyCompact(1500)} 🏛5 <span class="infotip">?<span class="infotip-text">技術、サイバー、宇宙産業の底上げに寄与します。</span></span></button>
        <button class="btn" onclick="eduAction('vocational')">🏭職業訓練 💰${formatMoneyCompact(1000)} <span class="infotip">?<span class="infotip-text">雇用改善と実体経済の底上げに寄与します。</span></span></button>`;
      break;
    case 'tech':
      title.textContent = '🔬 技術投資';
      content.innerHTML = `<button class="btn" onclick="techAction('ai')">🤖AI研究 💰${formatMoneyCompact(3000)} <span class="infotip">?<span class="infotip-text">技術株、GDP、サイバー、技術力の底上げに寄与します。</span></span></button>
        <button class="btn" onclick="techAction('quantum')">⚛量子技術 💰${formatMoneyCompact(5000)} 🏛10 <span class="infotip">?<span class="infotip-text">高難度だが技術力と産業革新への寄与が大きい投資です。</span></span></button>
        <button class="btn" onclick="techAction('green')">🌱グリーンテック 💰${formatMoneyCompact(2500)} <span class="infotip">?<span class="infotip-text">エネルギー効率と関連企業の評価改善に寄与します。</span></span></button>
        <button class="btn" onclick="techAction('space')">🚀宇宙開発 💰${formatMoneyCompact(4000)} 🏛12 <span class="infotip">?<span class="infotip-text">宇宙・防衛・技術の複合成長に寄与します。</span></span></button>`;
      break;
    case 'save':
      title.textContent = t.saveManager?.title || '💾 Save Manager';
      content.innerHTML = renderSaveManager();
      break;
    case 'settings':
      title.textContent = t.settings?.title || '⚙ Settings';
      content.innerHTML = renderSettingsPanel();
      break;
    case 'worldsetup':
      title.textContent = S.language === 'en' ? '🧭 World Creation' : '🧭 ワールド作成';
      content.innerHTML = renderWorldCreationPanel();
      break;
  }
  modal.classList.add('show');
}

function closeModal(){
  const modal = document.getElementById('actionModal');
  modal.classList.remove('show');
  modal.dataset.action = '';
  const modalCard = modal.querySelector('.modal');
  if(modalCard) modalCard.classList.remove('modal-world-country','modal-cyber','modal-finance','modal-company','modal-prefecture');
  disposeModalDom();
}

window.giveSubsidy = function(sector){
  if(S.money < 3000){ addEvent('❌ 資金不足','bad'); return; }
  if(S.polCap < 8){ addEvent('❌ 政治資本不足','bad'); return; }
  S.money -= 3000; S.polCap -= 8;
  
  // Check for unique material synergy
  let techRevolution = false;
  S.discoveredMaterials.forEach(dm=>{
    const mat = [...uniqueMaterials, ...spaceMaterials].find(m=>m.id===dm.id);
    if(mat && mat.effects[sector]){
      techRevolution = true;
      companies.filter(c=>c.sector===sector).forEach(c=>{
        c.price = Math.round(c.price * mat.effects[sector]);
      });
      addEvent(`✦ ${mat.name}を活用した技術革新！${sectors[sector].name}セクター大幅上昇！`,'good');
      // Related sector boost
      const relatedSectors = getRelatedSectors(sector);
      relatedSectors.forEach(rs=>{
        companies.filter(c=>c.sector===rs).forEach(c=>c.price=Math.round(c.price*1.04));
      });
    }
  });
  
  S.subsidyHeat[sector] = (S.subsidyHeat[sector] || 0) + rand(4, 9);
  if(!techRevolution){
    companies.filter(c=>c.sector===sector).forEach(c=>{
      c.price = Math.round(c.price * rand(1.008, 1.045));
    });
  }
  S.bubbleIndex += rand(2, 6);
  if(S.bubbleIndex > 50 && !S.bubbleActive && (S.totalWeeks || 0) >= (S.nextBubbleEligibleWeek || 260)){ S.bubbleSector = sector; triggerBubble(); }
  addEvent(`🏭 ${sectors[sector].name}に補助金3,000億投入`,'good');
  updateUI();
  openAction('subsidy');
};

function getRelatedSectors(sector){
  const relations = {
    tech:['telecom','defense'], auto:['mining','rubber'], energy:['mining','construction'],
    finance:['retail'], defense:['tech','construction'], pharma:['tech'],
    construction:['mining'], shipping:['energy'], food:['retail'],
    mining:['construction','auto'], telecom:['tech'], retail:['food','shipping'],
    space:['tech','defense','construction'], hospital:['pharma','tech']
  };
  return relations[sector] || [];
}

window.improveDiplomacy = function(i){
  if(S.polCap < 8){ addEvent('❌ 政治資本不足','bad'); return; }
  S.polCap -= 8;
  tradePartners[i].relation = Math.min(100, tradePartners[i].relation + 10);
  S.tradeDeals.filter(d=>d.partnerIdx===i).forEach(d=>{
    if(!d.isSell) d.cost *= 0.9;
    else d.cost *= 1.1; // better export prices
  });
  addEvent(`🌐 ${tradePartners[i].name}との関係改善 (${tradePartners[i].relation})`,'good');
  S.approval = Math.min(100, S.approval + 0.45);
  updateUI();
  openAction('diplomacy');
};

window.militaryAction = function(type){
  const costs = {equip:[3000,10,5],base:[5000,15,8],rd:[4000,8,3]};
  const [mc,pc,dp] = costs[type];
  if(S.money < mc){ addEvent('❌ 資金不足','bad'); return; }
  if(S.polCap < pc){ addEvent('❌ 政治資本不足','bad'); return; }
  const defenseGain = Math.round(dp * rand(0.85, 1.35));
  S.money -= mc; S.polCap -= pc; S.defPower += defenseGain;
  companies.filter(c=>c.sector==='defense').forEach(c=>c.price=Math.round(c.price*rand(type==='rd'?1.02:1.01, type==='rd'?1.09:1.07)));
  if(type==='base') companies.filter(c=>c.sector==='construction').forEach(c=>c.price=Math.round(c.price*rand(1.01,1.05)));
  if(type==='rd'){ companies.filter(c=>c.sector==='tech').forEach(c=>c.price=Math.round(c.price*rand(1.005,1.03))); S.cyberPower+=rand(0.4,1.4); S.techLevel += 0.4; }
  addEvent(`⚔ 軍事${type==='equip'?'装備更新':type==='base'?'基地拡張':'R&D'} 防衛力+${defenseGain}`,'good');
  updateUI();
  openAction('military');
};

window.eduAction = function(type){
  const costs = {uni:2000,stem:1500,vocational:1000};
  if(S.money < costs[type]){ addEvent('❌ 資金不足','bad'); return; }
  S.money -= costs[type];
  if(type==='stem'){ S.polCap-=5; companies.filter(c=>c.sector==='tech'||c.sector==='space').forEach(c=>c.price=Math.round(c.price*rand(1.005,1.03))); S.gdp*=rand(1.0006,1.0014); S.techLevel += rand(0.25,0.7); S.approval = Math.min(100, S.approval + 0.4); }
  else if(type==='uni'){ S.gdp *= rand(1.0004,1.001); S.techLevel += rand(0.15,0.4); S.approval = Math.min(100, S.approval + 0.55); }
  else { S.gdp *= rand(1.0002,1.0007); S.unemployment = Math.max(1.5, S.unemployment - rand(0.03,0.1)); S.approval = Math.min(100, S.approval + 0.25); }
  addEvent(`📚 教育投資実施 GDP成長`,'good');
  updateUI();
  openAction('education');
};

window.techAction = function(type){
  const costs = {ai:[3000,0,2],quantum:[5000,10,5],green:[2500,0,0],space:[4000,12,0]};
  const [mc,pc,cyber] = costs[type];
  if(S.money < mc){ addEvent('❌ 資金不足','bad'); return; }
  if(pc > 0 && S.polCap < pc){ addEvent('❌ 政治資本不足','bad'); return; }
  S.money -= mc; S.polCap -= pc; S.cyberPower += cyber;
  const boost = type==='quantum'?rand(1.02,1.08):type==='ai'?rand(1.01,1.05):rand(1.005,1.04);
  companies.filter(c=>c.sector==='tech').forEach(c=>c.price=Math.round(c.price*boost));
  if(type==='green') companies.filter(c=>c.sector==='energy').forEach(c=>c.price=Math.round(c.price*rand(1.01,1.04)));
  if(type==='space'){ S.defPower+=rand(1,3); companies.filter(c=>c.sector==='space').forEach(c=>c.price=Math.round(c.price*rand(1.01,1.05))); }
  S.techLevel += type==='quantum'?rand(0.8,1.8):type==='ai'?rand(0.4,1.1):rand(0.2,0.7);
  S.gdp *= type==='quantum'?rand(1.0008,1.0024):type==='ai'?rand(1.0006,1.0018):rand(1.0003,1.0012);
  addEvent(`🔬 技術投資(${type})完了`,'good');
  updateUI();
  openAction('tech');
};

// ============================================================
// BUBBLE
// ============================================================
function triggerBubble(){
  if((S.totalWeeks || 0) < (S.nextBubbleEligibleWeek || 260)) return false;
  S.bubbleActive = true;
  S.bubblePeak = S.bubbleIndex;
  S.nextBubbleEligibleWeek = Math.max(S.nextBubbleEligibleWeek || 0, (S.totalWeeks || 0) + 260);
  if(!S.bubbleSector){
    const sectorHits = {};
    companies.forEach(c=>{
      if(!sectorHits[c.sector]) sectorHits[c.sector] = 0;
      sectorHits[c.sector] += (c.price/c.base - 1) * 100;
    });
    S.bubbleSector = Object.entries(sectorHits).sort((a,b)=>b[1]-a[1])[0][0];
  }
  addEvent(`🎈 バブル発生！${sectors[S.bubbleSector].name}セクター過熱！日銀で抑制可能`,'bad');
  return true;
}

// ============================================================
// EVENTS
// ============================================================
const eventLog = document.getElementById('eventLog');
const majorEventBanner = document.getElementById('majorEventBanner');
function addEvent(msg, type){
  const div = document.createElement('div');
  div.className = 'event-msg' + (type==='bad'?' bad':type==='good'?' good':'');
  div.textContent = `[${S.year}年${S.week}週] ${msg}`;
  eventLog.prepend(div);
  ensureStateShape();
  S.eventHistory.unshift({label:`${S.year}年 第${S.week}週`, msg, type});
  if(S.eventHistory.length > 160) S.eventHistory.pop();
  if(eventLog.children.length > 20) eventLog.removeChild(eventLog.lastChild);
  setTimeout(()=>{ if(div.parentNode) div.style.opacity = '0.2'; }, 10000);
}

function showMajorEventBanner(text){
  majorEventBanner.textContent = text;
  majorEventBanner.classList.add('show');
  clearTimeout(showMajorEventBanner._timer);
  showMajorEventBanner._timer = setTimeout(() => majorEventBanner.classList.remove('show'), 5200);
}

function hideCinematicEvent(){
  const layer = document.getElementById('cinematicEvent');
  const smoke = document.getElementById('cinematicSmoke');
  if(!layer) return;
  layer.className = '';
  layer.setAttribute('aria-hidden', 'true');
  if(smoke) smoke.innerHTML = '';
}

function showCinematicEvent(options={}){
  const layer = document.getElementById('cinematicEvent');
  const badge = document.getElementById('cinematicEventBadge');
  const icon = document.getElementById('cinematicEventIcon');
  const title = document.getElementById('cinematicEventTitle');
  const subtitle = document.getElementById('cinematicEventSubtitle');
  const smoke = document.getElementById('cinematicSmoke');
  if(!layer || !badge || !icon || !title || !subtitle || !smoke) return;
  const tone = options.tone || 'victory';
  badge.textContent = options.badge || (tone === 'defeat' ? 'RETREAT' : 'VICTORY');
  icon.textContent = options.icon || (tone === 'defeat' ? '⚠' : '🏆');
  title.textContent = options.title || '';
  subtitle.textContent = options.subtitle || '';
  smoke.innerHTML = '';
  for(let i = 0; i < 16; i++){
    const puff = document.createElement('span');
    puff.className = 'smoke-puff';
    const angle = (-Math.PI / 2) + ((i - 7.5) / 15) * Math.PI * 1.28 + (Math.random() - 0.5) * 0.25;
    const dist = 110 + Math.random() * 170;
    puff.style.setProperty('--dx', `${Math.cos(angle) * dist}px`);
    puff.style.setProperty('--dy', `${Math.sin(angle) * dist - 42 - Math.random() * 46}px`);
    puff.style.setProperty('--scale', `${1.08 + Math.random() * 1.18}`);
    puff.style.setProperty('--alpha', `${0.26 + Math.random() * 0.36}`);
    puff.style.setProperty('--dur', `${2.6 + Math.random() * 1.8}s`);
    puff.style.setProperty('--delay', `${Math.random() * 0.45}s`);
    puff.style.width = `${72 + Math.random() * 86}px`;
    puff.style.height = puff.style.width;
    smoke.appendChild(puff);
  }
  for(let i = 0; i < 10; i++){
    const spark = document.createElement('span');
    spark.className = 'spark';
    const angle = (-Math.PI / 2) + ((i - 4.5) / 9) * Math.PI * 1.08 + (Math.random() - 0.5) * 0.35;
    const dist = 90 + Math.random() * 130;
    spark.style.setProperty('--dx', `${Math.cos(angle) * dist}px`);
    spark.style.setProperty('--dy', `${Math.sin(angle) * dist - 12 - Math.random() * 26}px`);
    spark.style.setProperty('--dur', `${1.1 + Math.random() * 1.1}s`);
    spark.style.setProperty('--delay', `${Math.random() * 0.35}s`);
    smoke.appendChild(spark);
  }
  clearTimeout(showCinematicEvent._timer);
  layer.className = `show ${tone}`;
  layer.setAttribute('aria-hidden', 'false');
  showCinematicEvent._timer = setTimeout(hideCinematicEvent, Math.max(2600, options.duration || 4200));
}

function triggerGameOverSequence(reason){
  if(S.gameOverPending) return;
  S.gameOverPending = true;
  setSpeed(0);
  showCinematicEvent({
    tone:'defeat',
    badge:'GAME OVER',
    icon:'☠',
    title:S.language === 'en' ? 'Government Collapse' : '政権崩壊',
    subtitle:reason || (S.language === 'en' ? 'Approval fell to zero and the cabinet lost control of the nation.' : '支持率が 0% に達し、内閣は国家運営を維持できなくなりました。'),
    duration:5200,
  });
  setTimeout(() => {
    resetGameState(S.language === 'en' ? 'Approval hit 0%, so the campaign restarted.' : '支持率が 0% になったため、最初からやり直しです。');
    S.gameOverPending = false;
  }, 4200);
}

function applyRegionalDisaster(type, regionKey, severity){
  const region = regions[regionKey];
  if(!region) return;
  const center = getRegionDisasterCenter(regionKey);
  const radius = 0.045 + severity * 0.04;
  const durationWeeks = Math.round(56 + severity * 132);
  region.disaster = {type, severity, weeksLeft:durationWeeks, totalWeeks:durationWeeks, x:center.x, y:center.y, radius, startedAt:Date.now()};
  const visual = getIncidentVisualProfile(type);
  spawnMapEffect({
    mode:'japan',
    x:center.x,
    y:center.y,
    radius,
    life:Math.round(18 + severity * 22),
    label:type,
    icon:visual.icon,
    color:visual.color,
  });
  companies.filter(company => company.region === regionKey).forEach(company => {
    const dist = worldDistance(company.mapX ?? region.cx, company.mapY ?? region.cy, center.x, center.y);
    if(dist > radius) return;
    const hit = clamp(1 - dist / Math.max(0.0001, radius), 0.18, 1);
    company.price = Math.round(company.price * (1 - severity * hit * rand(0.18, 0.42)));
    company.revenue = Math.max(120, Math.round(company.revenue * (1 - severity * hit * rand(0.08, 0.2))));
    company.favorability = Math.max(0, (company.favorability || 50) - severity * hit * 10);
  });
  S.publicWorks = S.publicWorks.filter(work => {
    const dist = worldDistance(work.x, work.y, center.x, center.y);
    if(dist > radius) return true;
    if(Math.random() < severity * clamp(1 - dist / radius, 0.2, 1) * 0.45){
      addEvent(`💥 ${regions[work.region]?.name || ''} の ${publicWorksCatalog[work.type]?.name || work.type} が ${type} で失われた`, 'bad');
      return false;
    }
    work.damage = Math.max(work.damage || 0, severity * clamp(1 - dist / radius, 0.2, 1));
    return true;
  });
  region.resources.forEach(resourceKey => {
    if(resources[resourceKey]) resources[resourceKey].amount *= Math.max(0.4, 1 - severity * 0.35);
  });
  addEvent(`🌋 ${region.name} で ${type} が発生。地域経済と供給網が大きく損傷`, 'bad');
}

const majorEventCatalog = [
  {
    id:'lehman',
    name:'世界金融危機',
    duration:20,
    headline:'世界金融危機が発生。信用収縮で実体経済が急減速',
    start(){
      companies.filter(c => ['finance','auto','retail','construction','shipping'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(0.55, 0.78)));
      S.gdp *= 0.968;
      S.stability = Math.max(0, S.stability - 10);
      S.businessCycle -= 0.22;
    },
    weekly(){
      S.gdp *= 0.9979;
      S.unemployment = Math.min(15, S.unemployment + 0.12);
      S.approval = Math.max(0, S.approval - 0.14);
      S.foreignUnrest += 4;
    }
  },
  {
    id:'pandemic',
    name:'世界的パンデミック',
    duration:24,
    headline:'感染症の世界流行で移動・観光・物流が混乱',
    start(){
      companies.filter(c => ['shipping','retail','food','auto','finance'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(0.6, 0.82)));
      companies.filter(c => ['hospital','pharma','tech'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(1.08, 1.22)));
      S.approval = Math.max(0, S.approval - 4);
      S.businessCycle -= 0.18;
    },
    weekly(){
      S.gdp *= 0.9982;
      S.unemployment = Math.min(15, S.unemployment + 0.08);
      S.stability = Math.max(0, S.stability - 0.45);
    }
  },
  {
    id:'ai_boom',
    name:'AI革命',
    duration:16,
    headline:'生成AI革命が始まり、技術・通信・宇宙産業に資金が集中',
    start(){
      companies.filter(c => ['tech','telecom','space'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(1.04, 1.12)));
      S.techLevel += 2;
      S.businessCycle += 0.08;
    },
    weekly(){
      S.gdp *= 1.00025;
      S.techLevel += 0.08;
      S.cyberPower += 0.04;
    }
  },
  {
    id:'energy_crisis',
    name:'エネルギー危機',
    duration:17,
    headline:'資源供給網が乱れ、原油・ガス・電力価格が急騰',
    start(){
      if(resources.oil) resources.oil.price *= 1.22;
      if(resources.gas) resources.gas.price *= 1.24;
      companies.filter(c => ['energy'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(1.05, 1.16)));
      companies.filter(c => ['shipping','auto','retail'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(0.68, 0.88)));
    },
    weekly(){
      S.inflation = Math.min(12, S.inflation + 0.16);
      S.stability = Math.max(0, S.stability - 0.25);
    }
  },
  {
    id:'yen_crisis',
    name:'円急変ショック',
    duration:12,
    headline:'通貨市場が大きく乱高下し、輸入業と金融市場が激しく動揺',
    start(){
      S.yenValue = Math.max(58, S.yenValue - rand(12, 22));
      companies.filter(c => ['retail','food','energy','finance'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(0.7, 0.88)));
      companies.filter(c => ['auto','tech','shipping'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(0.94, 1.06)));
    },
    weekly(){
      S.inflation = Math.min(12, S.inflation + 0.12);
      S.stability = Math.max(0, S.stability - 0.18);
    }
  },
  {
    id:'supply_chain',
    name:'世界供給網断裂',
    duration:14,
    headline:'海上物流と半導体供給が大きく途絶し、製造業が急減速',
    start(){
      companies.filter(c => ['auto','tech','shipping','construction'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(0.62, 0.82)));
      ['silicon','semiconductor','rareearth','lithium'].forEach(key => { if(resources[key]) resources[key].amount *= rand(0.45, 0.7); });
      S.gdp *= 0.975;
    },
    weekly(){
      S.unemployment = Math.min(16, S.unemployment + 0.08);
      S.gdp *= 0.9984;
    }
  },
  {
    id:'megaquake',
    name:'巨大地震と津波',
    duration:10,
    headline:'沿岸部で巨大地震と津波が発生し、地域産業と発電設備が停止',
    start(){
      const key = pick(['kanto','kinki','kyushu','tohoku','chubu']);
      applyRegionalDisaster('巨大地震・津波', key, rand(0.65, 0.95));
      companies.filter(c => c.sector === 'construction').forEach(c => c.price = Math.round(c.price * rand(1.04, 1.14)));
      S.gdp *= 0.972;
    },
    weekly(){
      S.stability = Math.max(0, S.stability - 0.22);
    }
  },
  {
    id:'commodity_crash',
    name:'資源価格暴落',
    duration:11,
    headline:'世界的な需要減退で資源価格が急落し、鉱業・エネルギー株が崩れる',
    start(){
      ['oil','gas','coal','copper','lithium','rareearth'].forEach(key => { if(resources[key]) resources[key].price *= rand(0.62, 0.8); });
      companies.filter(c => ['energy','mining'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(0.58, 0.8)));
    },
    weekly(){
      S.businessCycle -= 0.07;
    }
  },
  {
    id:'global_rearm',
    name:'世界的再軍備',
    duration:13,
    headline:'安全保障不安の高まりで防衛・サイバー関連に巨額資金が流入',
    start(){
      companies.filter(c => ['defense','tech','space'].includes(c.sector)).forEach(c => c.price = Math.round(c.price * rand(1.06, 1.18)));
      S.defPower += 3;
      S.businessCycle += 0.05;
    },
    weekly(){
      S.cyberPower += 0.06;
      S.gdp *= 1.00012;
    }
  }
];

// ============================================================
// TOOLTIP
// ============================================================
function hideTooltip(){ document.getElementById('tooltip').style.display = 'none'; }

function showTooltip(text, x, y){
  const tooltip = document.getElementById('tooltip');
  if(!tooltip || !text) return;
  tooltip.textContent = text;
  tooltip.style.display = 'block';
  tooltip.style.left = `${Math.min(window.innerWidth - tooltip.offsetWidth - 18, x + 14)}px`;
  tooltip.style.top = `${Math.min(window.innerHeight - tooltip.offsetHeight - 18, y + 14)}px`;
}

document.addEventListener('mouseover', event => {
  const target = event.target.closest('[data-tip]');
  if(!target) return;
  showTooltip(target.dataset.tip, event.clientX, event.clientY);
});

document.addEventListener('mousemove', event => {
  const target = event.target.closest('[data-tip]');
  if(!target) return;
  showTooltip(target.dataset.tip, event.clientX, event.clientY);
});

document.addEventListener('mouseout', event => {
  if(event.target.closest('[data-tip]')) hideTooltip();
});

let holdRepeatTimer = null;
let holdRepeatAction = null;
window.startRepeatAction = function(callbackName, value){
  if(typeof window[callbackName] !== 'function') return;
  holdRepeatAction = () => window[callbackName](value);
  holdRepeatAction();
  clearInterval(holdRepeatTimer);
  holdRepeatTimer = setInterval(() => {
    if(holdRepeatAction) holdRepeatAction();
  }, 120);
};

window.stopRepeatAction = function(){
  holdRepeatAction = null;
  clearInterval(holdRepeatTimer);
  holdRepeatTimer = null;
};

document.addEventListener('mouseup', ()=> window.stopRepeatAction());
document.addEventListener('mouseleave', ()=> window.stopRepeatAction());

function getMajorEventSectorModifier(sector){
  if(!S.activeMajorEvent) return 0;
  const id = S.activeMajorEvent.id;
  if(id === 'lehman'){
    if(['finance','auto','retail','construction'].includes(sector)) return -0.012;
    if(['hospital','food'].includes(sector)) return 0.003;
  }
  if(id === 'pandemic'){
    if(['shipping','retail','auto'].includes(sector)) return -0.01;
    if(['hospital','pharma','tech'].includes(sector)) return 0.009;
  }
  if(id === 'ai_boom'){
    if(['tech','telecom','space'].includes(sector)) return 0.012;
    if(['hospital'].includes(sector)) return 0.003;
  }
  if(id === 'energy_crisis'){
    if(['energy'].includes(sector)) return 0.01;
    if(['auto','shipping','retail'].includes(sector)) return -0.012;
  }
  if(id === 'yen_crisis'){
    if(['retail','food','energy','finance'].includes(sector)) return -0.012;
    if(['auto','tech','shipping'].includes(sector)) return 0.003;
  }
  if(id === 'supply_chain'){
    if(['auto','tech','shipping','construction'].includes(sector)) return -0.014;
    if(['hospital'].includes(sector)) return 0.002;
  }
  if(id === 'megaquake'){
    if(['construction'].includes(sector)) return 0.01;
    if(['energy','retail','shipping'].includes(sector)) return -0.01;
  }
  if(id === 'commodity_crash'){
    if(['energy','mining'].includes(sector)) return -0.015;
    if(['retail','auto'].includes(sector)) return 0.004;
  }
  if(id === 'global_rearm'){
    if(['defense','space','tech'].includes(sector)) return 0.013;
  }
  if(id === 'bio_outbreak'){
    if(['shipping','retail','food','finance','auto'].includes(sector)) return -0.02;
    if(['hospital','pharma'].includes(sector)) return 0.017;
  }
  return 0;
}

function startMajorEvent(eventDef){
  const duration = Math.max(52, Math.round(eventDef.duration * 5.5));
  S.activeMajorEvent = {id:eventDef.id, name:eventDef.name, weeksLeft:duration, duration};
  S.majorEventCooldown = duration + 64;
  eventDef.start();
  showMajorEventBanner(`重大イベント: ${eventDef.name}`);
  addEvent(`📣 ${eventDef.headline}`, eventDef.id === 'ai_boom' ? 'good' : 'bad');
}

function resolveNuclearProject(project){
  if(project.type === 'plant'){
    const chance = clamp(0.46 + S.techLevel * 0.018 + (resources.uranium?.amount || 0) / 10000 * 0.14, 0.2, 0.9);
    if(Math.random() < chance){
      S.nuclearPlants++;
      S.energyBonus += 5;
      S.techLevel += 0.8;
      companies.filter(c=>c.sector==='energy' || c.sector==='construction').forEach(c=>c.price=Math.round(c.price*rand(1.015,1.05)));
      addEvent(`☢ 原子力発電所${S.nuclearPlants}号機が完成`, 'good');
    } else {
      S.approval = Math.max(0, S.approval - 2.5);
      S.nuclear.failures++;
      addEvent('☢ 原発建設計画が遅延・頓挫した', 'bad');
    }
  } else {
    const chance = clamp(0.22 + S.techLevel * 0.012 + S.defPower * 0.003, 0.08, 0.72);
    if(Math.random() < chance){
      S.nukePower++;
      S.defPower += 24;
      S.nuclear.doctrine += 8;
      tradePartners.forEach(tp => tp.relation = Math.max(0, tp.relation - 6));
      companies.filter(c=>c.sector==='defense').forEach(c=>c.price=Math.round(c.price*rand(1.02,1.08)));
      addEvent(`🚀 核抑止計画が成功し、核戦力が ${S.nukePower} 基へ`, 'bad');
    } else {
      S.nuclear.failures++;
      S.approval = Math.max(0, S.approval - 4);
      addEvent('💥 核抑止計画に失敗し、国際的な批判が高まった', 'bad');
    }
  }
}

function getSectorCurrencyModifier(sector){
  const fx = 100 / Math.max(55, S.yenValue);
  const weakYen = fx - 1;
  if(['auto','tech','shipping','space','defense'].includes(sector)) return weakYen * 0.18;
  if(['retail','food','energy','hospital'].includes(sector)) return weakYen * -0.22;
  if(['finance'].includes(sector)) return weakYen * -0.08;
  return weakYen * 0.03;
}

function getResourceEventModifier(resourceKey){
  if(!S.activeMajorEvent) return 0;
  const id = S.activeMajorEvent.id;
  if(id === 'energy_crisis' && ['oil','gas','coal','uranium'].includes(resourceKey)) return 0.14;
  if(id === 'commodity_crash' && ['oil','gas','coal','copper','lithium','rareearth'].includes(resourceKey)) return -0.18;
  if(id === 'supply_chain' && ['silicon','semiconductor','rareearth','lithium','aluminum'].includes(resourceKey)) return 0.12;
  if(id === 'yen_crisis' && ['oil','gas','cotton','rubber','iron'].includes(resourceKey)) return 0.08;
  if(id === 'megaquake' && ['fish','timber','rice','oil','gas'].includes(resourceKey)) return 0.1;
  return 0;
}

function progressRegionalDisasters(){
  Object.entries(regions).forEach(([regionKey, region]) => {
    if(!region.disaster) return;
    if(typeof region.disaster.startedAt !== 'number') region.disaster.startedAt = Date.now();
    if(typeof region.disaster.totalWeeks !== 'number' || region.disaster.totalWeeks <= 0){
      region.disaster.totalWeeks = Math.max(1, region.disaster.weeksLeft || 1);
    }
    region.disaster.weeksLeft--;
    const severity = region.disaster.severity || 0.45;
    companies.filter(company => company.region === regionKey).forEach(company => {
      company.revenue = Math.max(120, Math.round(company.revenue * (1 - severity * 0.012)));
      company.price = Math.max(getCompanyDynamicFloor(company), Math.round(company.price * (1 - severity * 0.006)));
      company.sentiment = clamp((company.sentiment || 0) - severity * 0.025, -0.9, 0.9);
      company.favorability = clamp((company.favorability || 50) - severity * 0.12, 0, 100);
    });
    S.publicWorks.filter(work => work.region === regionKey && work.status === 'active').forEach(work => {
      work.damage = Math.max(work.damage || 0, severity * 0.32);
      work.operatingRate = Math.max(0, (work.operatingRate || 100) - severity * 4.2);
    });
    S.gdp *= 1 - severity * 0.00055;
    S.approval = Math.max(0, S.approval - severity * 0.015);
    S.stability = Math.max(0, S.stability - severity * 0.02);
    S.socialUnrest = clamp(S.socialUnrest + severity * 0.14, 0, 100);
    if(region.disaster.type === '大停電'){
      S.energyBonus = Math.max(0, S.energyBonus - 0.08);
    }
    if(region.disaster.type === '暴動'){
      S.socialUnrest = clamp(S.socialUnrest + 0.16, 0, 100);
    }
    if(region.disaster.weeksLeft > 0 && region.disaster.weeksLeft % 6 === 0){
      const visual = getIncidentVisualProfile(region.disaster.type);
      spawnMapEffect({
        mode:'japan',
        x:region.disaster.x,
        y:region.disaster.y,
        radius:region.disaster.radius,
        life:14,
        label:region.disaster.type,
        icon:visual.icon,
        color:visual.color,
      });
      addEvent(`⚠ ${region.name} の ${region.disaster.type} が継続中。企業と地域経済へ追加ダメージ`, 'bad');
    }
    if(region.disaster.weeksLeft > 0) return;
    const rebuildBoost = companies.filter(company => company.sector === 'construction').length ? rand(1.03, 1.08) : rand(1.01, 1.04);
    companies.filter(company => company.region === regionKey).forEach(company => {
      company.price = Math.round(company.price * rebuildBoost);
      company.revenue = Math.round(company.revenue * rand(1.01, 1.05));
    });
    addEvent(`🏗 ${region.name} の被災地復旧が進み、地域機能が回復`, 'good');
    region.disaster = null;
  });
}

function getPublicWorkScale(work){
  return clamp((work.operatingRate ?? 100) / 100, 0, 1.2);
}

function seedInitialPublicWorks(){
  if(S.publicWorks?.length) return;
  S.publicWorks = [
    {id:'seed-solar-kanto', type:'solar', x:0.645, y:0.455, region:'kanto', workers:100, status:'active', totalWeeks:0, weeksLeft:0, operatingRate:80, upkeep:publicWorksCatalog.solar.upkeep, seeded:true},
    {id:'seed-thermal-kinki', type:'thermal', x:0.515, y:0.505, region:'kinki', workers:100, status:'active', totalWeeks:0, weeksLeft:0, operatingRate:74, upkeep:publicWorksCatalog.thermal.upkeep, seeded:true},
    {id:'seed-hospital-kanto', type:'hospital', x:0.675, y:0.43, region:'kanto', workers:100, status:'active', totalWeeks:0, weeksLeft:0, operatingRate:72, upkeep:publicWorksCatalog.hospital.upkeep, seeded:true},
  ];
}

function normalizeSeededPublicWorks(){
  if(!Array.isArray(S.publicWorks)) return;
  const seededHospital = S.publicWorks.find(work => work.id === 'seed-hospital-kanto');
  if(seededHospital){
    seededHospital.x = 0.675;
    seededHospital.y = 0.43;
    seededHospital.region = 'kanto';
  }
}

function progressPublicWorks(){
  S.publicWorks.forEach(work => {
    if(work.status !== 'building') return;
    work.weeksLeft--;
    companies.filter(company => company.sector === 'construction').forEach(company => {
      company.revenue += 24;
      company.favorability = Math.min(100, company.favorability + 0.08);
    });
    if(work.weeksLeft > 0) return;
    work.status = 'active';
    work.weeksLeft = 0;
    work.operatingRate = work.operatingRate ?? 75;
    const config = publicWorksCatalog[work.type];
    companies.filter(company => ['construction','energy'].includes(company.sector)).forEach(company => {
      company.price = Math.round(company.price * rand(1.02, 1.06));
      company.revenue *= rand(1.01, 1.04);
      company.favorability = Math.min(100, company.favorability + 1.1);
    });
    if(work.type === 'hospital') S.approval = Math.min(100, S.approval + rand(0.8, 1.8));
    if(work.type === 'industrial') companies.filter(company => ['auto','tech','mining','construction'].includes(company.sector)).forEach(company => company.price = Math.round(company.price * rand(1.02, 1.08)));
    addEvent(`🏗 ${regions[work.region]?.name || ''} の ${config.name} が完成し、稼働を開始`, 'good');
  });
}

function progressTransportNetworks(){
  (S.transportNetworks || []).forEach(route => {
    if(route.status !== 'building') return;
    route.weeksLeft--;
    companies.filter(company => ['construction','shipping','auto','telecom'].includes(company.sector)).forEach(company => {
      company.revenue += 18;
      company.favorability = Math.min(100, company.favorability + 0.05);
    });
    if(route.weeksLeft > 0) return;
    route.status = 'active';
    route.weeksLeft = 0;
    route.operatingRate = route.operatingRate ?? 82;
    const config = transportNetworkCatalog[route.type];
    const endpoints = getTransportRouteEndpoints(route);
    [endpoints.from?.region, endpoints.to?.region].filter(Boolean).forEach(regionId => {
      companies.filter(company => company.region === regionId).forEach(company => {
        company.revenue *= rand(1.028, 1.072);
        company.price = Math.round(company.price * rand(1.015, 1.055));
        company.favorability = Math.min(100, company.favorability + 1.6);
      });
    });
    addEvent(`🚉 ${endpoints.from?.name || ''} と ${endpoints.to?.name || ''} の ${config?.name || '交通網'} が開通`, 'good');
  });
}

function getCompanyRevenueCeiling(company){
  const baseRevenue = Math.max(120, Number(company.baseRevenue || company.revenue || 120));
  const demandFactor = clamp((company.demand || company.baseDemand || 100) / 100, 0.72, 2.1);
  const techFactor = 1 + Math.max(0, (company.techDepth || 20) - 20) / 180 + Math.max(0, (company.techLevel || 20) - 20) / 240;
  const favorFactor = 1 + Math.max(0, (company.favorability || 50) - 50) / 220;
  const ownedShares = (S.ownedStocks?.[company.name] || 0);
  const ownershipRatio = clamp(ownedShares / Math.max(1, company.sharesOutstanding || 1), 0, 1);
  const ownershipFactor = 1 + Math.min(0.42, ownershipRatio * 0.55);
  const sectorFactor = ['tech','space','defense','telecom'].includes(company.sector) ? 1.18 : ['finance','construction','shipping'].includes(company.sector) ? 1.08 : 1;
  return Math.max(420, baseRevenue * demandFactor * techFactor * favorFactor * ownershipFactor * sectorFactor * 3.4);
}

function applyCompanyRevenueSupport(company, targetFactor=1, reaction=0.06){
  const ceiling = getCompanyRevenueCeiling(company);
  const baseRevenue = Math.max(120, Number(company.baseRevenue || company.revenue || 120));
  const targetRevenue = Math.min(ceiling, Math.max(120, baseRevenue * Math.max(0.72, targetFactor)));
  company.revenue += (targetRevenue - company.revenue) * clamp(reaction, 0.02, 0.18);
  if(company.revenue > ceiling){
    company.revenue = ceiling - Math.max(18, baseRevenue * 0.02);
  }
}

function applyTransportNetworkOperations(){
  let operationsCost = 0;
  let publicRevenue = 0;
  (S.transportNetworks || []).forEach(route => {
    if(route.status !== 'active') return;
    const config = transportNetworkCatalog[route.type];
    if(!config) return;
    const rateScale = clamp((route.operatingRate ?? 100) / 100, 0, 1.2);
    const upkeep = (config.upkeep || 0) * rateScale;
    operationsCost += upkeep;
    const endpoints = getTransportRouteEndpoints(route);
    const connectedIds = [endpoints.from?.id, endpoints.to?.id].filter(Boolean);
    const population = connectedIds.reduce((sum, prefectureId) => sum + (S.prefectureStats?.[prefectureId]?.population || 0), 0);
    const economicMass = connectedIds.reduce((sum, prefectureId) => sum + getPrefectureEconomicMass(prefectureId), 0);
    const distanceFactor = clamp(worldDistance(endpoints.from?.x || 0, endpoints.from?.y || 0, endpoints.to?.x || 0, endpoints.to?.y || 0) / 0.11, 0.75, 2.4);
    const users = Math.round((population * (route.type === 'rail' ? 0.24 : route.type === 'expressway' ? 0.18 : route.type === 'ferry' ? 0.12 : 0.09) + economicMass * (route.type === 'airlink' ? 54 : 36)) * rateScale * distanceFactor);
    const revenue = users * (route.type === 'rail' ? 0.022 : route.type === 'expressway' ? 0.016 : route.type === 'ferry' ? 0.018 : 0.026);
    route.weeklyUsers = Math.max(0, users);
    route.weeklyRevenue = Math.max(0, revenue);
    route.weeklyBalance = revenue - upkeep;
    route.monthlyUsers = route.weeklyUsers * 4;
    route.monthlyRevenue = route.weeklyRevenue * 4;
    route.monthlyBalance = route.weeklyBalance * 4;
    publicRevenue += revenue;
    S.money += revenue;
    connectedIds.forEach(prefectureId => {
      const stat = S.prefectureStats?.[prefectureId];
      if(!stat) return;
      stat.productivity = clamp(stat.productivity + config.productivity * 0.05 * rateScale, -35, 90);
      stat.appeal = clamp((stat.appeal || 50) + config.appeal * 0.038 * rateScale, 0, 100);
    });
    companies.filter(company => connectedIds.includes(company.prefecture)).forEach(company => {
      const productivityBoost = 1 + config.companyBoost * rateScale * (route.type === 'rail' ? 0.58 : route.type === 'airlink' ? 0.52 : 0.38);
      company.baseRevenue = Math.max(120, company.baseRevenue * (1 + config.companyBoost * rateScale * 0.0022));
      applyCompanyRevenueSupport(company, productivityBoost, 0.055 + config.companyBoost * 1.2);
      company.favorability = Math.min(100, company.favorability + config.companyBoost * 7 * rateScale);
    });
  });
  S.money -= operationsCost;
  return {cost:operationsCost, revenue:publicRevenue};
}

function updateEnergyBalance(){
  let production = S.nuclearPlants * 12;
  let demand = 20 + S.pop * 0.004 + S.gdp * 0.012 + S.techLevel * 0.46;
  S.publicWorks.forEach(work => {
    const config = publicWorksCatalog[work.type];
    if(!config || work.status !== 'active') return;
    const scale = getPublicWorkScale(work);
    production += (config.output || 0) * scale;
    demand += (config.energyDemand || 0) * scale;
    if(work.type === 'industrial') demand += (config.throughput || 0) * 0.2 * scale;
  });
  S.energy.production = production * (1 + S.energyBonus * 0.01);
  S.energy.demand = demand;
}

function applyPublicWorkOperations(){
  let operationsCost = 0;
  let publicRevenue = 0;
  const energyTightness = (S.energy.production - S.energy.demand) / Math.max(20, S.energy.demand);
  S.publicWorks.forEach(work => {
    const config = publicWorksCatalog[work.type];
    if(!config || work.status !== 'active') return;
    const scale = getPublicWorkScale(work);
    const upkeep = (config.upkeep || 0) * scale;
    const demandProfile = getPublicWorkDemandProfile(work);
    work.weeklyUsers = demandProfile.users;
    work.weeklyRevenue = demandProfile.revenue;
    work.weeklyBalance = demandProfile.balance;
    work.monthlyUsers = demandProfile.users * 4;
    work.monthlyRevenue = demandProfile.revenue * 4;
    work.monthlyBalance = demandProfile.balance * 4;
    operationsCost += upkeep;
    publicRevenue += demandProfile.revenue;
    S.money += demandProfile.revenue;
    const regionPrefectures = prefectures.filter(prefecture => prefecture.region === work.region);
    regionPrefectures.forEach(prefecture => {
      const stat = S.prefectureStats?.[prefecture.id];
      if(!stat) return;
      const baseBoost = work.type === 'industrial' ? 0.18 : work.type === 'hospital' ? 0.12 : 0.085;
      stat.productivity = clamp(stat.productivity + scale * baseBoost, -35, 90);
      stat.appeal = clamp((stat.appeal || 50) + scale * (work.type === 'hospital' ? 0.22 : work.type === 'industrial' ? 0.16 : 0.12), 0, 100);
    });
    if(work.type === 'industrial'){
      const industrialLift = 1 + scale * 0.058 + Math.max(0, energyTightness) * 0.045;
      companies.filter(company => company.region === work.region || ['auto','tech','mining','construction','shipping','telecom'].includes(company.sector)).forEach(company => {
        company.baseRevenue = Math.max(120, company.baseRevenue * (1 + scale * 0.0028));
        applyCompanyRevenueSupport(company, industrialLift, 0.075);
        company.favorability = Math.min(100, company.favorability + 0.22 * scale);
      });
      ['iron','copper','silicon','aluminum'].forEach(resourceKey => {
        const resource = resources[resourceKey];
        if(resource) resource.amount = Math.max(0, resource.amount - resource.max * 0.00034 * scale);
      });
      S.gdp *= 1 + scale * 0.00062;
    } else if(work.type === 'hospital'){
      S.approval = Math.min(100, S.approval + scale * 0.08);
      S.stability = Math.min(100, S.stability + scale * 0.08);
      companies.filter(company => company.region === work.region || ['hospital','pharma','retail'].includes(company.sector)).forEach(company => {
        company.baseRevenue = Math.max(120, company.baseRevenue * (1 + scale * 0.0016));
        applyCompanyRevenueSupport(company, 1 + scale * 0.032, 0.062);
        company.favorability = Math.min(100, company.favorability + 0.2 * scale);
      });
    } else if(['thermal','hydro','solar'].includes(work.type)){
      companies.filter(company => company.region === work.region || ['energy','construction','telecom','tech'].includes(company.sector)).forEach(company => {
        company.baseRevenue = Math.max(120, company.baseRevenue * (1 + scale * 0.0014));
        applyCompanyRevenueSupport(company, 1 + scale * 0.028, 0.056);
        company.favorability = Math.min(100, company.favorability + 0.12 * scale);
      });
    }
  });
  S.money -= operationsCost;
  return {cost:operationsCost, revenue:publicRevenue};
}

function getTradeExchangeFactor(partner){
  const pairShift = (partner.currencyRate || partner.baseCurrencyRate || 1) / Math.max(0.0001, partner.baseCurrencyRate || 1);
  const yenShift = 100 / Math.max(60, S.yenValue);
  return clamp(pairShift * yenShift, 0.58, 1.9);
}

function progressTradeRoutes(){
  let tradeNet = 0;
  S.tradeDeals.forEach(deal => {
    if(typeof deal.cooldown !== 'number') deal.cooldown = 0;
    if(deal.cooldown > 0){
      deal.cooldown--;
      return;
    }
    const partner = tradePartners[deal.partnerIdx];
    const exportable = getExportableResource(deal.resource);
    const actualResourceKey = deal.actualResource || exportable?.resource || deal.resource;
    const resource = resources[actualResourceKey];
    const exchange = getTradeExchangeFactor(partner);
    const cyberAdv = S.cyberPower > partner.cyberPower ? 1.08 : 0.96;
    const baseAmount = Math.max(1, (resource?.max || 10000) * (deal.pct / 100) * (deal.isSell ? 0.018 : 0.032));
    const travelWeeks = getCountryShippingWeeks(deal.partnerIdx);
    const priceBase = exportable?.price || deal.priceBase || resource?.price || 100;
    if(deal.isSell){
      if(resource && resource.amount < baseAmount){
        if(S.totalWeeks % 8 === 0) addEvent(`⚠ ${partner.name} 向け ${deal.resourceName} の輸出量を確保できない`, 'bad');
        deal.cooldown = 2;
        return;
      }
      if(resource) resource.amount = Math.max(0, resource.amount - baseAmount);
      const value = priceBase * baseAmount * 0.01 * exchange * cyberAdv * clamp(partner.relation / 60, 0.4, 1.6);
      S.tradeShipments.push({id:`ts-${Date.now()}-${Math.random().toString(36).slice(2,6)}`, partnerIdx:deal.partnerIdx, resourceKey:deal.resource, resourceName:deal.resourceName, actualResource:actualResourceKey, isSell:true, amount:baseAmount, value, totalWeeks:travelWeeks, weeksLeft:travelWeeks});
    } else {
      const value = priceBase * baseAmount * 0.01 * exchange * (cyberAdv > 1 ? 0.94 : 1.08) * clamp((120 - partner.relation) / 65, 0.5, 1.8);
      if(S.money < value){
        deal.cooldown = 2;
        return;
      }
      S.money -= value;
      tradeNet -= value;
      S.tradeShipments.push({id:`ts-${Date.now()}-${Math.random().toString(36).slice(2,6)}`, partnerIdx:deal.partnerIdx, resourceKey:deal.resource, resourceName:deal.resourceName, actualResource:actualResourceKey, isSell:false, amount:baseAmount, value, totalWeeks:travelWeeks, weeksLeft:travelWeeks});
    }
    deal.cooldown = Math.max(2, Math.round(travelWeeks * 0.55));
  });
  return tradeNet;
}

function progressTradeShipments(){
  let tradeNet = 0;
  S.tradeShipments.forEach(shipment => shipment.weeksLeft--);
  const delivered = S.tradeShipments.filter(shipment => shipment.weeksLeft <= 0);
  S.tradeShipments = S.tradeShipments.filter(shipment => shipment.weeksLeft > 0);
  delivered.forEach(shipment => {
    if(shipment.isSell){
      S.money += shipment.value;
      tradeNet += shipment.value;
    } else {
      const resource = resources[shipment.actualResource || shipment.resourceKey];
      if(resource) resource.amount = Math.min(resource.max * 2.6, resource.amount + shipment.amount);
      tradeNet -= shipment.value;
    }
  });
  return tradeNet;
}

function progressCompanyRoadmaps(){
  companies.forEach(company => {
    syncCompanyRoadmap(company);
    const stage = company.roadmap?.[company.roadmapIndex];
    if(!stage) return;
    const support = getCompanyAnnualSupport(company.name) / 52;
    const ownedRatio = clamp((S.ownedStocks[company.name] || 0) / Math.max(1, company.sharesOutstanding), 0, 1);
    const needRatios = company.needs.map(resourceKey => {
      const resource = resources[resourceKey];
      return resource?.max ? clamp(resource.amount / resource.max, 0, 1.1) : 1;
    });
    const resourceSupport = needRatios.length ? needRatios.reduce((sum, value) => sum + value, 0) / needRatios.length : 1;
    const cycleSupport = Math.max(0, S.businessCycle);
    const favorSupport = Math.max(0, (company.favorability || 50) - 50);
    const progressGain = clamp(
      0.12
      + S.techBudget * 0.026
      + S.educationBudget * 0.012
      + support * 0.00045
      + ownedRatio * 0.22
      + cycleSupport * 0.34
      + Math.max(0, resourceSupport - 0.74) * 0.4
      + favorSupport * 0.006,
      0.03, 1.3
    );
    company.techProgress += progressGain;
    if(company.techProgress < stage.threshold){
      persistCompanyRoadmapState(company);
      return;
    }
    company.techProgress -= stage.threshold;
    company.roadmapIndex++;
    company.techDepth += 3 + stage.boost * 16;
    company.base = Math.max(100, Math.round(company.base * (1 + stage.boost * 0.48)));
    company.price = Math.max(100, Math.round(company.price * (1 + stage.boost * 0.22)));
    company.revenue = Math.round(company.revenue * (1 + stage.boost * 0.5));
    company.favorability = Math.min(100, company.favorability + 2.5);
    persistCompanyRoadmapState(company);
    if(company.roadmapIndex >= company.roadmap.length || stage.boost >= 0.24 || stage.announce === 'major'){
      addEvent(`🧭 ${company.name} が技術節目「${stage.name}」を達成。${stage.note}`, 'good');
    }
  });
}

function progressForeignMarkets(){
  ensureForeignCompanies();
  S.foreignCompanies.forEach(company => {
    const partner = tradePartners[company.partnerIdx];
    if(!partner) return;
    const scarPenalty = (partner.nuclearScars || 0) * 0.0016;
    company.prevPrice = company.price;
    company.prevRevenue = company.revenue;
    const macroDrift = clamp(
      (partner.growth || 0) * 0.004
      + (partner.techLevel - 55) * 0.0007
      - Math.max(0, (partner.inflation || 2) - 3) * 0.002
      - Math.max(0, (partner.hostility || 0) - 55) * 0.0009
      - scarPenalty
      + rand(-0.012, 0.014),
      -0.08, 0.08
    );
    const warPenalty = S.activeWar && S.activeWar.partnerIdx === company.partnerIdx ? -0.05 : 0;
    company.revenue = Math.max(100, Math.round(company.revenue * (1 + macroDrift + warPenalty)));
    const revenueGrowth = company.prevRevenue > 0 ? (company.revenue / company.prevRevenue - 1) : 0;
    const desiredBase = Math.max(100, company.base * clamp(1 + revenueGrowth * 0.22 + macroDrift * 0.12 - Math.max(0, company.price / Math.max(100, company.base) - 1.08) * 0.12, 0.982, 1.014));
    company.base = Math.max(100, Math.round(lerp(company.base, desiredBase, 0.3)));
    const dynamicFloor = getCompanyDynamicFloor(company);
    const softCeiling = getCompanySoftCeiling(company);
    const meanTarget = company.base * (1 + revenueGrowth * 0.24 + macroDrift * 0.24 + Math.sin(S.totalWeeks * 0.11 + (company.floorWaveSeed || 0)) * 0.02);
    let candidate = company.price
      + (meanTarget - company.price) * 0.24
      + rand(-company.base * 0.028, company.base * 0.028)
      + Math.sin(S.totalWeeks * 0.31 + (company.floorWaveSeed || 0)) * company.base * 0.012;
    if(candidate > softCeiling){
      const overshoot = candidate - softCeiling;
      candidate = softCeiling + overshoot * 0.18 + Math.sin(S.totalWeeks * 0.45 + (company.floorWaveSeed || 0)) * company.base * 0.04 + rand(-company.base * 0.03, company.base * 0.02);
      if(Math.random() < 0.08 + overshoot / Math.max(1, softCeiling) * 0.18) candidate *= rand(0.87, 0.96);
    }
    if(candidate < dynamicFloor){
      candidate = dynamicFloor + Math.abs(candidate - dynamicFloor) * 0.14 + rand(0, company.base * 0.018);
    }
    company.price = clamp(Math.round(candidate), dynamicFloor, softCeiling * 1.22);
    maybeTriggerStockBaseShock(company, {silent:true, triggerRatio:2.7});
    company.hist.push(company.price);
    company.revenueHist.push(company.revenue);
    if(company.hist.length > 1040) company.hist.shift();
    if(company.revenueHist.length > 1040) company.revenueHist.shift();
  });
}

function getWarStateCountry(){
  if(!S.activeWar) return null;
  return worldCountries.find(country => country.id === S.activeWar.countryId) || null;
}

function getAllMilitaryUnitKeys(){
  return ['infantry','tanks','fighters','ships','drones','spies'];
}

function getUnitKeys(){
  return ['infantry','tanks','fighters','ships','drones'];
}

function cloneUnits(units={}){
  return getUnitKeys().reduce((acc, key) => {
    acc[key] = Math.max(0, Math.round(units[key] || 0));
    return acc;
  }, {});
}

function countUnits(units={}){
  return getUnitKeys().reduce((sum, key) => sum + (units[key] || 0), 0);
}

function getUnitsPower(units={}, side='jp', partner=null){
  const catalog = getMilitaryCatalog();
  const raw = getUnitKeys().reduce((sum, key) => sum + (units[key] || 0) * (catalog[key]?.power || 0), 0);
  if(side === 'enemy' && partner){
    return raw * (1 + partner.techLevel * 0.004 + partner.cyberPower * 0.0014 + partner.hostility * 0.001);
  }
  return raw * (1 + S.techLevel * 0.0022 + S.cyberPower * 0.0012 + S.nationalScore * 0.01);
}

function canViewWarIntel(countryId){
  const country = getWorldCountry(countryId);
  const partner = getWorldCountryPartner(country);
  if(!partner) return true;
  return partner.relation >= 55 || S.activeWar?.countryId === countryId || (partner.spyIntelWeeks || 0) > 0;
}

function getDisplayMilitaryIntel(countryId){
  if(countryId === 'japan'){
    const units = cloneUnits(S.militaryInventory || {});
    return {units, power:getUnitsPower(units, 'jp') + Math.max(0, S.defPower * 0.4)};
  }
  if(!canViewWarIntel(countryId)) return null;
  return getCountryMilitaryIntel(countryId);
}

function getCountryMilitaryIntel(countryId){
  const country = getWorldCountry(countryId);
  const partner = getWorldCountryPartner(country);
  if(!partner) return null;
  const units = createEnemyWarReserves(partner);
  return {units, power:getUnitsPower(units, 'enemy', partner) + partner.nukePower * 40 + partner.economy * 5};
}

function getSpyMissionTravelWeeks(partnerIdx){
  return Math.max(2, Math.round(getCountryShippingWeeks(partnerIdx) * 0.82));
}

function countActiveSpyMissions(countryId){
  return (S.spyMissions || []).filter(mission => mission.countryId === countryId).length;
}

function getSpyIntelStatus(countryId){
  const country = getWorldCountry(countryId);
  const partner = getWorldCountryPartner(country);
  if(!partner) return null;
  return {
    weeksLeft:partner.spyIntelWeeks || 0,
    alert:partner.spyAlert || 0,
    activeMissions:countActiveSpyMissions(countryId),
  };
}

function getSpyMissionSuccessChance(partner, qty){
  const cyberEdge = (S.cyberPower || 0) - (partner?.cyberPower || 0);
  const techEdge = (S.techLevel || 0) - (partner?.techLevel || 0);
  const companyLift = companies.filter(company => ['tech','telecom','defense','finance'].includes(company.sector)).reduce((sum, company) => sum + Math.max(0, (company.favorability || 50) - 50), 0);
  const quantityLift = Math.log2(Math.max(1, qty) + 1) * 0.08;
  const alertPenalty = (partner?.spyAlert || 0) * 0.008;
  const chance = 0.38 + cyberEdge * 0.004 + techEdge * 0.003 + companyLift / 6000 + quantityLift - alertPenalty;
  return clamp(chance, 0.12, 0.9);
}

window.sendSpyMission = function(countryId, qty=1){
  const country = getWorldCountry(countryId);
  const partner = getWorldCountryPartner(country);
  if(!country || !partner){ addEvent('❌ その国には諜報潜入できません', 'bad'); return; }
  qty = qty === 'all' ? (S.militaryInventory.spies || 0) : Math.max(1, Math.round(qty || 1));
  const available = S.militaryInventory.spies || 0;
  qty = Math.min(qty, available);
  if(qty <= 0){ addEvent('❌ 諜報班がいません', 'bad'); return; }
  const missionCost = Math.round(180 + qty * 24 + partner.cyberPower * 0.9);
  const missionPol = Math.max(8, Math.round(6 + qty * 0.8));
  if(S.money < missionCost){ addEvent('❌ 諜報作戦の資金が不足しています', 'bad'); return; }
  if(S.polCap < missionPol){ addEvent('❌ 諜報作戦の政治資本が不足しています', 'bad'); return; }
  S.money -= missionCost;
  S.polCap -= missionPol;
  S.militaryInventory.spies -= qty;
  const travelWeeks = getSpyMissionTravelWeeks(country.partnerIdx);
  S.spyMissions.push({
    id:`spy-${Date.now()}-${Math.random().toString(36).slice(2,6)}`,
    countryId,
    partnerIdx:country.partnerIdx,
    qty,
    totalWeeks:travelWeeks,
    weeksLeft:travelWeeks,
    successChance:getSpyMissionSuccessChance(partner, qty),
  });
  partner.spyAlert = clamp((partner.spyAlert || 0) + qty * 0.12, 0, 30);
  companies.filter(company => ['tech','telecom','defense','finance'].includes(company.sector)).forEach(company => {
    company.favorability = Math.min(100, (company.favorability || 50) + 0.35);
    company.price = Math.max(100, Math.round(company.price * rand(1.002, 1.015)));
  });
  addEvent(`🕵 ${country.name} へ諜報班 x${qty} を潜入派遣。到着まで ${travelWeeks}週`, 'good');
  updateUI();
};

function resolveSpyMission(mission){
  const country = getWorldCountry(mission.countryId);
  const partner = getWorldCountryPartner(country);
  if(!partner) return;
  const success = Math.random() < mission.successChance;
  const returnQty = Math.max(0, Math.round(mission.qty * (success ? rand(0.7, 1.0) : rand(0.15, 0.5))));
  S.militaryInventory.spies = (S.militaryInventory.spies || 0) + returnQty;
  if(success){
    const intelWeeks = 18 + mission.qty * 3 + Math.round(Math.max(0, S.cyberPower - partner.cyberPower) * 0.08);
    partner.spyIntelWeeks = Math.max(partner.spyIntelWeeks || 0, intelWeeks);
    partner.spyAlert = Math.max(0, (partner.spyAlert || 0) - mission.qty * 0.18);
    const rewardMoney = Math.round((160 + partner.economy * 9 + partner.techLevel * 4) * mission.qty * rand(0.85, 1.28));
    S.money += rewardMoney;
    partner.relation = Math.max(0, partner.relation - Math.max(1, Math.round(mission.qty * 0.18)));
    addEvent(`🕵 ${country.name} への潜入成功。軍事情報を奪取し ${formatMoneyCompact(rewardMoney)} を回収`, 'good');
  } else {
    const penalty = Math.round((140 + partner.cyberPower * 2.1) * mission.qty * rand(0.7, 1.2));
    S.money = Math.max(0, S.money - penalty);
    S.approval = Math.max(0, S.approval - Math.min(2.8, mission.qty * 0.08));
    partner.relation = Math.max(0, partner.relation - Math.max(2, Math.round(mission.qty * 0.35)));
    partner.hostility = Math.min(100, (partner.hostility || 0) + Math.round(mission.qty * 0.5));
    partner.spyAlert = clamp((partner.spyAlert || 0) + mission.qty * 0.55, 0, 40);
    addEvent(`💥 ${country.name} で諜報班が露見。${formatMoneyCompact(penalty)} の損失と外交悪化`, 'bad');
  }
}

function progressSpyMissions(){
  if(!Array.isArray(S.spyMissions) || !S.spyMissions.length){
    tradePartners.forEach(partner => {
      if(partner.spyIntelWeeks > 0) partner.spyIntelWeeks--;
      if(partner.spyAlert > 0) partner.spyAlert = Math.max(0, partner.spyAlert - 0.08);
    });
    return;
  }
  S.spyMissions.forEach(mission => {
    mission.weeksLeft--;
  });
  const completed = S.spyMissions.filter(mission => mission.weeksLeft <= 0);
  S.spyMissions = S.spyMissions.filter(mission => mission.weeksLeft > 0);
  completed.forEach(resolveSpyMission);
  tradePartners.forEach(partner => {
    if(partner.spyIntelWeeks > 0) partner.spyIntelWeeks--;
    if(partner.spyAlert > 0) partner.spyAlert = Math.max(0, partner.spyAlert - 0.08);
  });
}

function getWarTravelWeeks(partnerIdx){
  const logisticsLift = companies.filter(company => ['shipping','defense','telecom'].includes(company.sector)).reduce((sum, company) => sum + ((company.favorability || 50) - 50), 0);
  const relayBoost = S.space?.orbitalRelays ? S.space.orbitalRelays * 0.08 : 0;
  const modifier = clamp(1 - logisticsLift / 2400 - S.infraLevel * 0.03 - relayBoost, 0.58, 1.12);
  return Math.max(2, Math.round(getCountryShippingWeeks(partnerIdx) * modifier));
}

function getEnemyMilitaryGrowth(partner){
  return 1 + Math.max(0, S.totalWeeks / 52) * 0.035 + Math.max(0, (partner.growth || 0) * 0.018);
}

function createEnemyWarReserves(partner){
  const growth = getEnemyMilitaryGrowth(partner);
  return {
    infantry:Math.round((12000 + partner.economy * 140 + partner.stability * 26) * growth),
    tanks:Math.round((10000 + partner.economy * 96 + partner.techLevel * 34) * growth),
    fighters:Math.round((10000 + partner.techLevel * 62 + partner.economy * 40) * growth),
    ships:Math.round((10000 + partner.economy * 54 + partner.stability * 18) * growth),
    drones:Math.round((10000 + partner.cyberPower * 48 + partner.techLevel * 24) * growth),
  };
}

function createEmptyWarUnits(){
  return {infantry:0, tanks:0, fighters:0, ships:0, drones:0};
}

function mergeUnits(target, source){
  getUnitKeys().forEach(key => {
    target[key] = Math.max(0, Math.round((target[key] || 0) + (source[key] || 0)));
  });
}

function subtractUnits(target, source){
  getUnitKeys().forEach(key => {
    target[key] = Math.max(0, Math.round((target[key] || 0) - (source[key] || 0)));
  });
}

function getTotalWarPresence(units, dispatches){
  return countUnits(units) + dispatches.reduce((sum, dispatch) => sum + countUnits(dispatch.units), 0);
}

function pushWarLog(text, type=''){
  if(!S.activeWar) return;
  S.activeWar.logs = S.activeWar.logs || [];
  S.activeWar.logs.unshift({week:`${S.year}年${S.week}週`, text, type});
  if(S.activeWar.logs.length > 16) S.activeWar.logs.length = 16;
}

function spawnEnemyWarDispatch(war){
  const partner = tradePartners[war.partnerIdx];
  if(!partner) return false;
  const batch = createEmptyWarUnits();
  const priorities = ['infantry','tanks','fighters','drones','ships'];
  let sent = 0;
  priorities.forEach(key => {
    if(sent >= 3) return;
    const available = war.enemyReserves[key] || 0;
    if(available <= 0) return;
    const qty = Math.max(1, Math.round(Math.min(available, available * rand(0.08, 0.18))));
    batch[key] = qty;
    war.enemyReserves[key] -= qty;
    sent++;
  });
  if(countUnits(batch) <= 0) return false;
  war.enemyDispatches.push({side:'enemy', units:batch, weeksLeft:war.travelWeeks, totalWeeks:war.travelWeeks, returning:false});
  pushWarLog(`${partner.name} が部隊を前線へ輸送開始`, 'bad');
  return true;
}

function resolveWarOutcome(winner='jp', reason=''){
  const war = S.activeWar;
  if(!war) return;
  const country = getWorldCountry(war.countryId);
  const partner = tradePartners[war.partnerIdx];
  if(winner === 'jp'){
    if(!S.warVictories.includes(war.countryId)) S.warVictories.push(war.countryId);
    const spoils = Math.round(2800 + partner.gdpUsd * 600 + partner.economy * 40);
    const gdpDamage = 8;
    const populationLoss = rand(1, 8);
    const stabilityDamage = 14;
    const relationFloor = 8;
    S.money += spoils;
    partner.relation = Math.max(relationFloor, partner.relation);
    partner.economy = Math.max(8, partner.economy - 12);
    partner.stability = Math.max(0, partner.stability - stabilityDamage);
    partner.gdpUsd = Math.max(0.12, partner.gdpUsd * (1 - gdpDamage / 100));
    partner.gdp = scalePartnerGdp(partner.gdpUsd);
    partner.population = Math.max(1, partner.population - populationLoss);
    addEvent(`🏆 ${country.name} との戦争に勝利。当地企業への投資が解禁${reason ? ` / ${reason}` : ''}`, 'good');
    pushWarLog(`${country.name} に勝利`, 'good');
    openWarVictoryModal(country, {spoils, investment:true, gdpDamage, populationLoss, stabilityDamage, relationFloor});
  } else {
    S.approval = Math.max(0, S.approval - 6);
    S.stability = Math.max(0, S.stability - 6);
    partner.relation = Math.max(0, partner.relation - 4);
    addEvent(`⚠ ${country.name} との戦争に敗北${reason ? ` / ${reason}` : ''}`, 'bad');
    openWarDefeatModal(country, partner, reason);
  }
  S.activeWar = null;
}

window.sendWarUnits = function(unitType, qty){
  const war = S.activeWar;
  if(!war) return;
  const available = S.militaryInventory[unitType] || 0;
  if(available <= 0){ addEvent('❌ その兵種の予備戦力がありません', 'bad'); return; }
  if(qty === 'all') qty = available;
  qty = Math.max(1, Math.round(qty || 1));
  qty = Math.min(qty, available);
  const payload = createEmptyWarUnits();
  payload[unitType] = qty;
  S.militaryInventory[unitType] -= qty;
  war.jpDispatches.push({side:'jp', units:payload, weeksLeft:war.travelWeeks, totalWeeks:war.travelWeeks, returning:false});
  pushWarLog(`日本が ${getMilitaryCatalog()[unitType]?.name || unitType} を ${qty} 送軍`, 'good');
  addEvent(`🚚 ${getMilitaryCatalog()[unitType]?.name || unitType} を ${qty} 前線へ輸送開始`, '');
  renderPanel();
  updateUI();
};

window.returnWarUnits = function(){
  const war = S.activeWar;
  if(!war) return;
  if(countUnits(war.jpFront) <= 0){ addEvent('ℹ 前線に戻す兵がいません', ''); return; }
  const payload = cloneUnits(war.jpFront);
  war.jpFront = createEmptyWarUnits();
  war.jpDispatches.push({side:'jp', units:payload, weeksLeft:war.travelWeeks, totalWeeks:war.travelWeeks, returning:true});
  pushWarLog('前線部隊に帰投命令', '');
  addEvent('↩ 前線部隊を帰投させています', '');
  renderPanel();
};

window.fightWarBattle = function(){
  const war = S.activeWar;
  if(!war) return;
  const partner = tradePartners[war.partnerIdx];
  if(countUnits(war.jpFront) <= 0){ addEvent('❌ 前線の日本軍がいません', 'bad'); return; }
  if(countUnits(war.enemyFront) <= 0){ addEvent('❌ まだ敵軍が到着していません', 'bad'); return; }
  const jpPower = getUnitsPower(war.jpFront, 'jp') + S.defPower * 0.25 + S.cyberPower * 0.08;
  const enemyPower = getUnitsPower(war.enemyFront, 'enemy', partner) + partner.nukePower * 3;
  const delta = (jpPower - enemyPower) / Math.max(50, enemyPower);
  const jpLossFactor = clamp(0.12 - delta * 0.07 + rand(0.02, 0.08), 0.04, 0.38);
  const enemyLossFactor = clamp(0.12 + delta * 0.08 + rand(0.02, 0.08), 0.04, 0.42);
  getUnitKeys().forEach(key => {
    const jpLoss = Math.min(war.jpFront[key], Math.round((war.jpFront[key] || 0) * jpLossFactor));
    const enemyLoss = Math.min(war.enemyFront[key], Math.round((war.enemyFront[key] || 0) * enemyLossFactor));
    war.jpFront[key] -= jpLoss;
    war.enemyFront[key] -= enemyLoss;
  });
  war.warScore = (war.warScore || 0) + delta * 12;
  if(delta >= 0){
    partner.economy = Math.max(8, partner.economy - Math.max(0.4, delta * 1.6));
    partner.stability = Math.max(0, partner.stability - Math.max(0.6, delta * 2.2));
    pushWarLog(`日本軍が会戦で優勢。戦況 ${war.warScore.toFixed(1)}`, 'good');
    addEvent(`⚔ 会戦勝利。${partner.name} 軍を押し返した`, 'good');
  } else {
    S.approval = Math.max(0, S.approval - Math.abs(delta) * 1.4);
    S.stability = Math.max(0, S.stability - Math.abs(delta) * 1.2);
    pushWarLog(`敵軍が会戦で優勢。戦況 ${war.warScore.toFixed(1)}`, 'bad');
    addEvent(`⚠ 会戦劣勢。${partner.name} 軍に押し込まれた`, 'bad');
  }
  if(getTotalWarPresence(war.enemyFront, war.enemyDispatches) + countUnits(war.enemyReserves) <= 0){
    resolveWarOutcome('jp', '敵戦力を壊滅させた');
  } else if(getTotalWarPresence(war.jpFront, war.jpDispatches) + countUnits(S.militaryInventory) <= 0 && countUnits(war.enemyFront) > 0){
    resolveWarOutcome('enemy', '自軍戦力が尽きた');
  }
  renderPanel();
  updateUI();
};

window.worsenDiplomacy = function(index){
  const partner = tradePartners[index];
  if(!partner) return;
  if(S.polCap < 6){ addEvent('❌ 政治資本不足', 'bad'); return; }
  S.polCap -= 6;
  partner.relation = Math.max(0, partner.relation - 10);
  partner.hostility = Math.min(100, (partner.hostility || 0) + 10);
  addEvent(`🔥 ${partner.name} への対抗措置で関係が悪化 (${partner.relation})`, 'bad');
  renderPanel();
  updateUI();
};

window.declareWar = function(countryId){
  const country = getWorldCountry(countryId);
  const partner = getWorldCountryPartner(country);
  if(!country || !partner || country.id === 'japan') return;
  if(S.activeWar){ addEvent('❌ すでに戦争中です', 'bad'); return; }
  if(partner.relation > 35){ addEvent('❌ 友好度が高すぎて宣戦できません。まず敵対工作で関係を悪化させてください', 'bad'); return; }
  if(S.money < 14000 || S.polCap < 40){ addEvent('❌ 戦争を始める資金か政治資本が不足しています', 'bad'); return; }
  S.money -= 14000;
  S.polCap -= 40;
  S.activeWar = {
    countryId,
    partnerIdx:country.partnerIdx,
    jpFront:createEmptyWarUnits(),
    enemyFront:createEmptyWarUnits(),
    jpDispatches:[],
    enemyDispatches:[],
    enemyReserves:createEnemyWarReserves(partner),
    travelWeeks:getWarTravelWeeks(country.partnerIdx),
    warScore:0,
    enemyDispatchCooldown:1,
    enemyOccupation:0,
    battleReadyShownAt:0,
    logs:[],
  };
  partner.hostility = Math.min(100, (partner.hostility || 0) + 18);
  S.currentPanel = 'world';
  S.worldFocusCountry = countryId;
  storeCurrentView();
  S.currentMapMode = 'world';
  loadViewForMode('world');
  pushWarLog(`${country.name} と開戦。部隊派遣を開始せよ`, 'bad');
  addEvent(`⚔ ${country.name} と開戦。部隊を選んで送軍してください`, 'bad');
  updateMapHud();
  renderPanel();
  updateUI();
};

function progressWarFront(){
  if(!S.activeWar) return;
  const war = S.activeWar;
  const partner = tradePartners[war.partnerIdx];
  const preBattleReady = countUnits(war.jpFront) > 0 && countUnits(war.enemyFront) > 0;
  S.money -= 700 + countUnits(war.jpFront) * 16 + war.jpDispatches.reduce((sum, dispatch) => sum + countUnits(dispatch.units) * 6, 0);
  war.enemyDispatchCooldown = Math.max(0, (war.enemyDispatchCooldown || 0) - 1);
  if(war.enemyDispatchCooldown <= 0 && countUnits(war.enemyReserves) > 0){
    if(spawnEnemyWarDispatch(war)) war.enemyDispatchCooldown = Math.max(2, Math.round(war.travelWeeks * 0.8));
  }
  [...war.jpDispatches, ...war.enemyDispatches].forEach(dispatch => dispatch.weeksLeft--);
  war.jpDispatches = war.jpDispatches.filter(dispatch => {
    if(dispatch.weeksLeft > 0) return true;
    if(dispatch.returning) mergeUnits(S.militaryInventory, dispatch.units);
    else mergeUnits(war.jpFront, dispatch.units);
    pushWarLog(dispatch.returning ? '日本軍が本土へ帰還' : '日本軍が前線へ到着', dispatch.returning ? '' : 'good');
    return false;
  });
  war.enemyDispatches = war.enemyDispatches.filter(dispatch => {
    if(dispatch.weeksLeft > 0) return true;
    if(!dispatch.returning) mergeUnits(war.enemyFront, dispatch.units);
    pushWarLog(dispatch.returning ? '敵軍が後退' : `${partner.name} 軍が前線へ到着`, 'bad');
    return false;
  });
  if(countUnits(war.enemyFront) > 0 && countUnits(war.jpFront) <= 0 && war.jpDispatches.length <= 0){
    war.enemyOccupation = (war.enemyOccupation || 0) + 1;
    S.approval = Math.max(0, S.approval - 0.35);
    S.stability = Math.max(0, S.stability - 0.32);
    if(war.enemyOccupation > 7) resolveWarOutcome('enemy', '敵軍が前線を制圧した');
  } else {
    war.enemyOccupation = Math.max(0, (war.enemyOccupation || 0) - 1);
  }
  if(countUnits(war.enemyReserves) + getTotalWarPresence(war.enemyFront, war.enemyDispatches) <= 0){
    resolveWarOutcome('jp', '敵軍の後続が尽きた');
  }
  const postBattleReady = countUnits(war.jpFront) > 0 && countUnits(war.enemyFront) > 0;
  if(postBattleReady && !preBattleReady){
    war.battleReadyShownAt = S.totalWeeks;
    openWarBattleCommand(true);
  }
}

// ============================================================
// GAME TICK
// ============================================================
function gameTick(options={}){
  if(!S.started) return;
  invalidateRuntimeCaches('tick');
  S.polCapMax = S.sandboxMode ? 6000 : 400;
  if(!S.sandboxMode) S.polCap = Math.min(S.polCap, S.polCapMax);
  S.prevMoney=S.money;
  S.prevPolCap=S.polCap;
  S.prevPop=S.pop;
  S.prevGdp=S.gdp;
  S.prevApproval=S.approval;
  S.prevTechLevel=S.techLevel;
  S.prevYenValue=S.yenValue;
  S.weeklyBreakdown = {tax:0,budget:0,trade:0,dividend:0,space:0,interest:0,principal:0,companySupport:0,publicRevenue:0,operations:0,socialSecurity:0,healthcare:0,childcare:0,administration:0,adjustments:0};
  if((S.totalWeeks || 0) - (S.securityProfile?.lastIntegrityCheckWeek || 0) >= 52){
    verifyRuntimeIntegrity('tick-audit');
    if(S.securityProfile) S.securityProfile.lastIntegrityCheckWeek = S.totalWeeks || 0;
  }
  if(((S.totalWeeks || 0) % 4) === 0) auditEconomyIntegrity('tick');
  progressMapEffects();

  S.week++;
  S.totalWeeks++;
  let annualDividendPaid = 0;

  if(S.week > 52){
    S.week = 1;
    S.year++;
    S.polCap = Math.min(S.polCapMax, S.polCap + (S.sandboxMode ? 24 : 12));
    tradePartners.forEach(partner => {
      const scarLevel = partner.nuclearScars || 0;
      const ideologyAxis = clamp(S.ideologyScore / 100, -1, 1);
      const ideologyAffinity = getCountryIdeologyPreference(partner.id);
      const ideologyDrift = clamp(0.42 - Math.abs(ideologyAxis - ideologyAffinity) * 0.68 + (S.ideologyDrift || 0) * 0.01, -0.62, 0.46);
      partner.gdpUsd *= 1 + (partner.growth || 0) / 100 + rand(-0.01, 0.01);
      if(scarLevel > 0){
        partner.gdpUsd *= Math.max(0.7, 1 - scarLevel * 0.0038);
        partner.population = Math.max(1, partner.population - scarLevel * 0.04);
        partner.stability = clamp(partner.stability - scarLevel * 0.06, 0, 100);
        partner.nuclearScars = Math.max(0, scarLevel * 0.94 - 0.2);
      }
      partner.gdp = scalePartnerGdp(partner.gdpUsd);
      partner.cyberPower = clamp(partner.cyberPower + rand(-0.5, 1.4), 30, 160);
      partner.techLevel = clamp(partner.techLevel + rand(-0.2, 1.2), 20, 160);
      partner.stability = clamp(partner.stability + rand(-1.4, 1.8) - (partner.hostility || 0) * 0.01, 20, 100);
      partner.relation = clamp(partner.relation + rand(-2, 2) - Math.max(0, (partner.hostility || 0) - 55) * 0.02 + ideologyDrift, 0, 100);
      partner.hostility = clamp((partner.hostility || 0) + rand(-2, 2), 0, 100);
      partner.currencyRate = clamp(partner.currencyRate * (1 + ((partner.inflation || 2) - S.inflation) * 0.004 + rand(-0.03, 0.03)), partner.baseCurrencyRate * 0.72, partner.baseCurrencyRate * 1.35);
      partner.currencyHist.push(partner.currencyRate);
      if(partner.currencyHist.length > 1040) partner.currencyHist.shift();
    });
    if(S.lastDividendYear < S.year){
      let domesticDividend = 0;
      companies.forEach(company => {
        const ownedRatio = clamp((S.ownedStocks[company.name] || 0) / Math.max(1, company.sharesOutstanding), 0, 1);
        domesticDividend += company.revenue * company.margin * 0.22 * ownedRatio;
      });
      let foreignDividend = 0;
      ensureForeignCompanies();
      S.foreignCompanies.forEach(company => {
        const ownedRatio = clamp((S.foreignOwnedStocks[company.id] || 0) / Math.max(1, company.sharesOutstanding), 0, 1);
        foreignDividend += company.revenue * company.margin * 0.16 * ownedRatio;
      });
      if(domesticDividend + foreignDividend > 0){
        annualDividendPaid = domesticDividend + foreignDividend;
        S.money += annualDividendPaid;
    addEvent(`💴 年次配当を受領。国内 ${formatMoneyCompact(Math.round(domesticDividend))} / 海外 ${formatMoneyCompact(Math.round(foreignDividend))}`, 'good');
      }
      S.lastDividendYear = S.year;
    }
    addEvent(`📘 ${S.year}年 国家白書: GDP ${S.gdp.toFixed(0)}兆 / 支持率 ${S.approval.toFixed(1)}% / 円指数 ${S.yenValue.toFixed(1)}`, S.stability >= 60 ? 'good' : 'bad');
  }
  S.weeklyBreakdown.dividend = annualDividendPaid;

  if(S.activeMajorEvent){
    if(S.activeMajorEvent.id === 'bio_outbreak'){
      S.gdp *= 0.9976;
      S.unemployment = Math.min(16, S.unemployment + 0.08);
      S.stability = Math.max(0, S.stability - 0.32);
      S.activeMajorEvent.weeksLeft--;
      if(S.activeMajorEvent.weeksLeft <= 0){
        addEvent(`🌤 特殊パンデミック「${S.activeMajorEvent.name}」が収束に向かっている`, 'bad');
        S.activeMajorEvent = null;
      }
    } else {
      const eventDef = majorEventCatalog.find(entry => entry.id === S.activeMajorEvent.id);
      if(eventDef){
        eventDef.weekly();
        S.activeMajorEvent.weeksLeft--;
        if(S.activeMajorEvent.weeksLeft <= 0){
          addEvent(`🌤 重大イベント「${eventDef.name}」が収束に向かっている`, eventDef.id === 'ai_boom' || eventDef.id === 'global_rearm' ? 'good' : '');
          S.activeMajorEvent = null;
        }
      }
    }
  } else if(S.majorEventCooldown <= 0 && Math.random() < 0.004){
    startMajorEvent(pick(majorEventCatalog));
  } else {
    S.majorEventCooldown = Math.max(0, S.majorEventCooldown - 1);
  }

  progressWarFront();
  progressSpyMissions();
  progressRegionalDisasters();
  progressPublicWorks();
  progressMilitaryProjects();

  if(S.researchProjects.length){
    const completedResearch = [];
    S.researchProjects.forEach(project => {
      project.weeksLeft--;
      if(project.weeksLeft <= 0) completedResearch.push(project);
    });
    S.researchProjects = S.researchProjects.filter(project => project.weeksLeft > 0);
    completedResearch.forEach(resolveResearchProject);
  }

  if(S.space.mission){
    S.space.mission.weeksLeft--;
    if(S.space.mission.weeksLeft <= 0) resolveSpaceMission();
  }

  if(S.nuclear.projects.length){
    const completed = [];
    S.nuclear.projects.forEach(project => {
      project.weeksLeft--;
      if(project.weeksLeft <= 0) completed.push(project);
    });
    S.nuclear.projects = S.nuclear.projects.filter(project => project.weeksLeft > 0);
    completed.forEach(resolveNuclearProject);
  }

  const taxPressure = Math.max(0, S.taxRate - 18) * 0.0034;
  const taxRelief = Math.max(0, 18 - S.taxRate) * 0.002;
  S.businessCyclePhase += 0.08 + rand(-0.018, 0.018);
  S.businessCycle = clamp(
    S.businessCycle * 0.9 +
    Math.sin(S.businessCyclePhase) * 0.045 +
    rand(-0.02, 0.02) +
    (S.infraBudget + S.educationBudget > 16 ? 0.01 : 0) +
    taxRelief -
    taxPressure -
    Math.max(0, S.inflation - 4) * 0.008,
    -0.72, 0.72
  );

  Object.keys(S.subsidyHeat).forEach(key => {
    S.subsidyHeat[key] = Math.max(0, S.subsidyHeat[key] * 0.94 - 0.18);
    if(S.subsidyHeat[key] < 0.2) delete S.subsidyHeat[key];
  });

  const taxMultiplier = clamp(S.taxRate / 18, 0.45, 1.85);
  const taxIncome = (S.gdp * 0.006 + S.pop * 0.03 + Math.max(0, S.businessCycle) * 110) * taxMultiplier;
  const budgetCost = (S.militaryBudget + S.educationBudget + S.techBudget + S.infraBudget) * 22;
  S.money += taxIncome - budgetCost;
  S.weeklyBreakdown.tax = taxIncome;
  S.weeklyBreakdown.budget = budgetCost;

  const supportEntries = getBudgetSupportEntries();
  let companySupportCost = 0;
  supportEntries.forEach(entry => {
    const company = getCompanyByName(entry.company);
    if(!company) return;
    const weeklySupport = entry.amount / 52;
    companySupportCost += weeklySupport;
    S.money -= weeklySupport;
    company.revenue += weeklySupport * 0.42;
    company.techProgress += weeklySupport * 0.0007;
    persistCompanyRoadmapState(company);
    company.favorability = Math.min(100, (company.favorability || 50) + 0.04 + weeklySupport / 6000);
    company.price = Math.max(100, Math.round(company.price * (1 + Math.min(0.02, weeklySupport / Math.max(80000, company.revenue))))); 
  });
  S.weeklyBreakdown.companySupport = companySupportCost;

  if(S.techBudget === 0) S.gdp *= 0.9989;
  if(S.techBudget < 3) S.gdp *= 0.9993;
  S.techLevel += 0.01 + S.techBudget * 0.015 + S.educationBudget * 0.011 + S.space.level * 0.012 + S.discoveredMaterials.length * 0.004 - Math.max(0, S.inflation - 4) * 0.009;
  S.techLevel = clamp(S.techLevel, 0, 140);
  S.gdp *= (1 + S.techBudget * 0.000016 + S.educationBudget * 0.00002 + S.infraLevel * 0.000022 + S.energyBonus * 0.00003 + Math.max(0, S.businessCycle) * 0.00022);
  updateEnergyBalance();
  const publicOps = applyPublicWorkOperations();
  S.weeklyBreakdown.operations = publicOps.cost || 0;
  S.weeklyBreakdown.publicRevenue = publicOps.revenue || 0;
  updateEnergyBalance();
  if(S.space.level > 0){
    const spaceIncome = S.space.resources * 0.065 + S.space.factories * 36 + (S.space.orbitalRelays || 0) * 18;
    S.money += spaceIncome;
    S.gdp *= (1 + S.space.level * 0.00003 + S.space.factories * 0.00008);
    S.weeklyBreakdown.space = spaceIncome;
  }

  let weeklyInterest = 0;
  let weeklyPrincipalPaid = 0;
  S.loans.forEach(loan => {
    const interest = loan.amount * loan.rate / 52;
    weeklyInterest += interest;
    const principal = Math.min(loan.amount, loan.weeklyPrincipal || 0);
    weeklyPrincipalPaid += principal;
    loan.amount = Math.max(0, loan.amount - principal);
  });
  S.loans = S.loans.filter(loan => loan.amount > 1);
  S.money -= (weeklyInterest + weeklyPrincipalPaid);
  S.weeklyBreakdown.interest = weeklyInterest;
  S.weeklyBreakdown.principal = weeklyPrincipalPaid;

  if(S.nuclearPlants > 0 && resources.uranium){
    resources.uranium.amount = Math.max(0, resources.uranium.amount - S.nuclearPlants * 2.2);
    if(resources.uranium.amount <= 0){
      S.energyBonus = Math.max(0, S.energyBonus - 2);
      if(S.totalWeeks % 10 === 0) addEvent('⚠ ウラン不足で原発出力低下', 'bad');
    }
  }

  Object.entries(resources).forEach(([key, resource]) => {
    Object.entries(regions).forEach(([regionKey, region]) => {
      if(region.resources.includes(key) && region.developed > 0){
        const halted = region.disaster && region.disaster.weeksLeft > 0;
        const prod = halted ? 0 : resource.max * 0.00068 * region.developed * 0.28;
        resource.amount += prod;
      }
    });
    if(resource.domestic) resource.amount = Math.max(0, resource.amount - resource.max * 0.0006);
  });

  let tradeBalance = progressTradeRoutes();
  tradeBalance += progressTradeShipments();
  S.weeklyBreakdown.trade = tradeBalance;

  companies.forEach(company => {
    company.needs.forEach(resourceKey => {
      if(resources[resourceKey]) resources[resourceKey].amount = Math.max(0, resources[resourceKey].amount - resources[resourceKey].max * 0.00012);
    });
  });

  S.yenValue = clamp(
    S.yenValue +
    ((100 + tradeBalance / 900 - Math.max(0, S.inflation - 2.2) * 3.2 + S.techLevel * 0.12 + getAverageRelation() * 0.08 - getTotalDebt() / Math.max(1, S.gdp * 20)) - S.yenValue) * 0.05 +
    rand(-0.7, 0.7),
    55, 160
  );

  Object.entries(resources).forEach(([key, resource]) => {
    if(!resource.price) return;
    const supplyRatio = clamp(resource.amount / Math.max(1, resource.max), 0, 1.35);
    const fundamental = resource.basePrice * (1 + Math.min(0.6, S.totalWeeks / 1040 * 0.14));
    const eventModifier = getResourceEventModifier(key);
    const shortageLift = (0.58 - supplyRatio) * 0.24;
    const drift = (fundamental - resource.price) * 0.05;
    const noise = resource.basePrice * rand(-0.014, 0.014);
    const candidate = resource.price + drift + fundamental * shortageLift * 0.04 + resource.price * eventModifier * 0.04 + noise;
    const minBound = fundamental * (S.activeMajorEvent && ['commodity_crash'].includes(S.activeMajorEvent.id) ? 0.35 : 0.55);
    const maxBound = fundamental * (S.activeMajorEvent && ['energy_crisis','supply_chain'].includes(S.activeMajorEvent.id) ? 2.45 : 1.85);
    resource.price = clamp(candidate, minBound, maxBound);
    resource.priceHist.push(resource.price);
    if(resource.priceHist.length > 1040) resource.priceHist.shift();
  });

  S.foreignUnrest = Math.max(0, S.foreignUnrest + (Math.random() - 0.53) * 2.1 + (S.activeMajorEvent ? 0.18 : 0));
  if(S.foreignUnrest > 18){
    companies.forEach(company => company.price = Math.round(company.price * rand(1.0002, 1.004)));
    if(S.totalWeeks % 20 === 0) addEvent('💹 海外不安で日本株へ逃避資金が流入', 'good');
  }

  if(Math.random() < 0.012){
    const attacker = pick(tradePartners.filter(partner => partner.cyberPower > 35));
    if(attacker){
      const attackGap = attacker.cyberPower - S.cyberPower;
      const success = Math.random() < clamp(0.12 + attackGap * 0.006, 0.05, 0.62);
      if(success){
        const loss = Math.round(rand(600, 2400));
        S.money = Math.max(0, S.money - loss);
        attacker.relation = Math.max(0, attacker.relation - 2);
      addEvent(`💥 ${attacker.name} 由来のサイバー攻撃で ${formatMoneyCompact(loss)} の損失`, 'bad');
      } else if(S.totalWeeks % 10 === 0){
        addEvent(`🛡 ${attacker.name} からのサイバー攻撃を防御`, 'good');
      }
    }
  }

  const energyGap = (S.energy.production - S.energy.demand) / Math.max(20, S.energy.demand);
  S.socialUnrest = clamp(
    S.socialUnrest * 0.88
    + Math.max(0, 58 - S.approval) * 0.18
    + Math.max(0, 52 - S.stability) * 0.16
    + Math.max(0, S.inflation - 4.4) * 1.4
    + Math.max(0, S.unemployment - 6.2) * 1.2
    + Math.max(0, -energyGap) * 10
    + (S.activeMajorEvent ? 2.6 : 0)
    + (S.briberyExposure || 0) * 0.05
    + rand(-1.6, 1.4),
    0, 100
  );
  S.ideologyDrift *= 0.94;
  const workScale = S.publicWorks.reduce((acc, work) => {
    if(work.status !== 'active') return acc;
    const scale = getPublicWorkScale(work);
    if(work.type === 'industrial') acc.industrial += scale;
    else if(work.type === 'hospital') acc.hospital += scale;
    else if(['thermal','hydro','solar'].includes(work.type)) acc.power += scale;
    return acc;
  }, {industrial:0, hospital:0, power:0});
  let avgGrowth = 0;
  let avgRevenueChange = 0;
  companies.forEach(company => {
    company.prevPrice = company.price;
    company.prevRevenue = company.revenue;
    const needRatios = company.needs.map(resourceKey => {
      const resource = resources[resourceKey];
      if(!resource) return 1;
      return resource.max ? clamp(resource.amount / resource.max, 0, 1.2) : 1;
    });
    const availability = needRatios.length ? needRatios.reduce((sum, value) => sum + value, 0) / needRatios.length : 1;
    const missingCount = needRatios.filter(value => value < 0.03).length;
    const coveredCount = needRatios.filter(value => value >= 0.08).length;
    const healthyCount = needRatios.filter(value => value >= 0.28).length;
    const coverageRatio = needRatios.length ? coveredCount / needRatios.length : 1;
    const healthyRatio = needRatios.length ? healthyCount / needRatios.length : 1;
    const shortagePenalty = missingCount * 0.016 + Math.max(0, 0.14 - availability) * 0.12 + Math.max(0, 0.34 - coverageRatio) * 0.032;
    const materialSupport =
      Math.max(0, coverageRatio - 0.18) * 0.0065 +
      Math.max(0, healthyRatio - 0.3) * 0.0045 +
      Math.max(0, availability - 0.5) * 0.0075;
    let sectorBudgetBoost = 0;
    if(company.sector === 'defense') sectorBudgetBoost += S.militaryBudget * 0.0007;
    if(['tech','space'].includes(company.sector)) sectorBudgetBoost += S.techBudget * 0.00062 + (S.space.orbitalRelays || 0) * 0.0024;
    if(company.sector === 'telecom') sectorBudgetBoost += S.techBudget * 0.0008 + (S.space.orbitalRelays || 0) * 0.0036;
    if(company.sector === 'construction') sectorBudgetBoost += S.infraBudget * 0.00105;
    if(company.sector === 'finance') sectorBudgetBoost += Math.max(0, S.businessCycle) * 0.008 + S.infraBudget * 0.00016;
    if(company.sector === 'energy') sectorBudgetBoost += S.energyBonus * 0.0005;
    if(['hospital','pharma'].includes(company.sector)) sectorBudgetBoost += S.educationBudget * 0.00055;
    let publicWorkBoost = 0;
    if(['auto','tech','mining','construction','shipping','finance'].includes(company.sector)) publicWorkBoost += workScale.industrial * 0.0085;
    if(['hospital','pharma','food'].includes(company.sector)) publicWorkBoost += workScale.hospital * 0.007;
    if(['energy','construction'].includes(company.sector)) publicWorkBoost += workScale.power * 0.0055;
    const techBoost = S.techLevel * 0.00065;
    const ideologyBoost = getCompanyIdeologyModifier(company);
    const eventBoost = getMajorEventSectorModifier(company.sector);
    const cycleBoost = S.businessCycle * (['construction','auto','retail','finance'].includes(company.sector) ? 0.12 : ['hospital','pharma','food'].includes(company.sector) ? 0.06 : 0.08);
    const fxBoost = getSectorCurrencyModifier(company.sector);
    const subsidyBoost = (S.subsidyHeat[company.sector] || 0) * 0.0012;
    const fatiguePenalty = Math.max(0, (S.subsidyHeat[company.sector] || 0) - 12) * 0.0015;
    const disasterPenalty = regions[company.region]?.disaster ? regions[company.region].disaster.severity * 0.08 : 0;
    const support = getCompanyAnnualSupport(company.name) / 52;
    const supportBoost = support / Math.max(6000, company.base * 2.6);
    const energyFactor = ['auto','tech','space','construction','mining','shipping','energy'].includes(company.sector) ? energyGap * 0.09 : energyGap * 0.045;
    const favorBoost = ((company.favorability || 50) - 50) * 0.0018;
    const roadmapBoost = (company.techDepth || 0) * 0.00045;
    const unrestPenalty = S.socialUnrest * 0.0012 * (['finance','retail','auto','construction'].includes(company.sector) ? 1.15 : 0.78);
    const companyRhythm = Math.sin(S.totalWeeks * (0.08 + Math.abs(company.idleBias || 0) * 0.018) + (company.floorWaveSeed || 0)) * 0.0048;
    const startupRatio = Math.max(0, 78 - S.totalWeeks) / 78;
    const startupRevenueGuard = startupRatio * 0.011;
    const startupPriceGuard = startupRatio * 0.018;
    const operatingBaseline = Math.max(0, availability - 0.18) * 0.01 + Math.max(0, coverageRatio - 0.22) * 0.006;
    const demandDrift = 1 + sectorBudgetBoost + publicWorkBoost + techBoost + ideologyBoost + roadmapBoost + cycleBoost + fxBoost + eventBoost * 0.5 + subsidyBoost + supportBoost * 0.4 + favorBoost + energyFactor + materialSupport * 0.22 + operatingBaseline + startupRevenueGuard - shortagePenalty * 0.82 - fatiguePenalty * 0.78 - disasterPenalty - unrestPenalty * 0.82 + companyRhythm + (company.idleBias || 0) * 0.0022 + rand(-0.013, 0.015);
    company.revenue = Math.max(120, Math.round(company.revenue * clamp(demandDrift, 0.988, 1.076)));
    if(startupRatio > 0 && availability > 0.18){
      const startupRevenueFloor = Math.max(120, company.baseRevenue * (0.93 + startupRatio * 0.08));
      if(company.revenue < startupRevenueFloor){
        company.revenue = Math.round(startupRevenueFloor + rand(-company.baseRevenue * 0.008, company.baseRevenue * 0.008));
      }
    }
    const revenueGrowth = company.prevRevenue > 0 ? (company.revenue / company.prevRevenue - 1) : 0;
    avgRevenueChange += revenueGrowth;
    company.revenueHist.push(company.revenue);
    if(company.revenueHist.length > 1040) company.revenueHist.shift();
    const revenueMomentum = company.revenue / Math.max(1, getHistoricReference(company.revenueHist, 26)) - 1;
    const priceStretchNow = company.price / Math.max(100, company.base || 100);
    company.base = Math.max(100, Math.round(company.base * clamp(
      1
      + revenueGrowth * 0.24
      + revenueMomentum * 0.05
      + sectorBudgetBoost * 0.12
      + ideologyBoost * 0.16
      + publicWorkBoost * 0.22
      + materialSupport * 0.05
      + techBoost * 0.12
      + roadmapBoost * 0.2
      + supportBoost * 0.14
      + favorBoost * 0.12
      + energyFactor * 0.08
      + fxBoost * 0.08
      + eventBoost * 0.08
      - shortagePenalty * 0.22
      - Math.max(0, materialSupport - 0.03) * 0.02
      - fatiguePenalty * 0.24
      - disasterPenalty * 0.24
      - Math.max(0, priceStretchNow - 1.08) * 0.12
      - Math.max(0, S.socialUnrest - 35) * 0.0008,
      0.975, 1.018
    )));
    company.sentiment = clamp(
      company.sentiment * 0.7
      + revenueGrowth * 0.92
      + publicWorkBoost * 0.45
      + ideologyBoost * 0.34
      + materialSupport * 0.04
      + sectorBudgetBoost * 0.26
      + eventBoost * 0.55
      + energyFactor * 0.22
      + supportBoost * 0.18
      - shortagePenalty * 0.6
      - fatiguePenalty * 0.55
      - disasterPenalty * 0.42
      - S.socialUnrest * 0.0032
      + rand(-company.volatility * 1.2, company.volatility * 1.2),
      -0.7, 0.95
    );
    const overvaluation = Math.max(0, company.price / Math.max(100, company.base) - 1.16);
    const originalStretch = company.price / Math.max(100, company.floorBase || company.base);
    const stretchPenalty = Math.max(0, originalStretch - 1.25);
    const crashRiskRaw = (S.activeMajorEvent ? 0.018 : 0) + (S.businessCycle < -0.24 ? 0.012 : 0) + (fatiguePenalty > 0.01 ? 0.018 : 0) + (availability < 0.08 ? 0.014 : 0) + Math.max(0, originalStretch - 2.1) * 0.1 + S.socialUnrest * 0.0012 + stretchPenalty * 0.01;
    const crashRisk = crashRiskRaw * (1 - startupRatio * 0.68);
    const passiveBubbleLift = clamp((S.bubbleIndex || 0) / 320 * 0.16, 0, 0.16);
    const activeSectorBubbleLift = S.bubbleActive && company.sector === S.bubbleSector ? 0.05 + Math.max(0, S.bubbleIndex - 60) * 0.0024 : 0;
    const bubbleLift = passiveBubbleLift + activeSectorBubbleLift;
    company.recoveryLag = Math.max(0, (company.recoveryLag || 0) - 1);
    const recoveryPenalty = company.recoveryLag > 0 ? 0.06 + Math.min(0.18, company.recoveryLag / 520) : 0;
    const dynamicFloor = getCompanyDynamicFloor(company);
    const softCeiling = getCompanySoftCeiling(company) * (1 + bubbleLift * 0.1) * (1 - recoveryPenalty * 0.22);
    const patternSignal = company.sentiment * 0.55 + revenueMomentum * 0.7 + S.businessCycle * 0.45 - shortagePenalty * 0.6 - S.socialUnrest * 0.004;
    maybeRotateCompanyPattern(company, patternSignal);
    const pattern = getCompanyPatternForce(company);
    const wave = Math.sin(S.totalWeeks * 0.34 + (company.floorWaveSeed || 0)) * company.base * 0.006;
    const earlyMarketDrag = S.totalWeeks < 52 ? startupPriceGuard + rand(-0.004, 0.005) + (company.idleBias || 0) * 0.0014 : 0;
    const meanTarget = company.base * (
      1
      + company.sentiment * 0.08
      + revenueGrowth * 0.42
      + revenueMomentum * 0.08
      + publicWorkBoost * 0.08
      + ideologyBoost * 0.28
      + sectorBudgetBoost * 0.06
      + materialSupport * 0.035
      + supportBoost * 0.05
      + bubbleLift * 0.45
      + earlyMarketDrag
      - shortagePenalty * 0.12
      - disasterPenalty * 0.12
      - overvaluation * 0.18
      - recoveryPenalty
    );
    const jaggedAmplitude = company.base * (0.011 + company.volatility * 0.95 + Math.max(0, overvaluation) * 0.014 + Math.max(0, -patternSignal) * 0.008);
    let candidate = company.price
      + (meanTarget - company.price) * 0.21
      + pattern.shape * jaggedAmplitude
      + pattern.bias * company.base * 0.014
      + rand(-company.base * (0.012 + company.volatility * 0.55 + recoveryPenalty * 0.08 + Math.max(0, S.socialUnrest - 36) * 0.0009), company.base * (0.011 + company.volatility * 0.35))
      + wave;
    if(pattern.age > 0.82 && Math.random() < 0.14 + Math.abs(pattern.bias) * 0.08){
      candidate += pattern.bias * company.base * rand(0.012, 0.034);
    }
    if(candidate > softCeiling){
      const overshoot = candidate - softCeiling;
      candidate = softCeiling
        + overshoot * rand(0.08, 0.24)
        + triangleWave((S.totalWeeks / Math.max(4, company.patternLength || 8)) + (company.floorWaveSeed || 0)) * company.base * 0.04
        + rand(-company.base * 0.04, company.base * 0.024);
      if(Math.random() < 0.12 + overshoot / Math.max(1, softCeiling) * 0.22) candidate *= rand(0.72, 0.95);
    }
    if(Math.random() < crashRisk){
      if(originalStretch > 2.75){
        candidate *= rand(0.2, 0.68);
      } else if(originalStretch > 2.1){
        candidate *= rand(0.56, 0.88);
      } else if(originalStretch > 1.55){
        candidate *= rand(0.78, 0.94);
      } else {
        candidate *= rand(0.92, 0.985);
      }
    }
    if(candidate < dynamicFloor){
      candidate = dynamicFloor
        + Math.abs(candidate - dynamicFloor) * 0.18
        + triangleWave((S.totalWeeks * 0.18) + (company.floorWaveSeed || 0)) * company.base * 0.005
        + rand(0, company.base * 0.014);
    }
    if(startupRatio > 0){
      const startupFloor = dynamicFloor + Math.max(0, company.base - dynamicFloor) * (0.22 * startupRatio);
      if(candidate < startupFloor){
        candidate = startupFloor + rand(-company.base * 0.01, company.base * 0.012);
      }
    }
    const revenueAnchor = Math.max(
      dynamicFloor,
      company.revenue * (company.revenueScaleHint || 0.018)
      + (company.techDepth || 0) * 7
      + (company.favorability || 50) * 4
      + Math.max(0, company.base * 0.08)
    );
    const revenueGap = clamp((candidate - revenueAnchor) / Math.max(400, revenueAnchor), -1, 1);
    candidate += Math.max(0, -revenueGap) * company.base * 0.05;
    candidate -= Math.max(0, revenueGap) * company.base * 0.14;
    company.price = clamp(Math.round(candidate), dynamicFloor, softCeiling * 1.26);
    maybeTriggerStockBaseShock(company, {triggerRatio:2.85});
    company.favorability = clamp((company.favorability || 50) + 0.03 + support / 20000 + publicWorkBoost * 1.1 + Math.max(0, S.businessCycle) * 0.05 - Math.max(0, S.taxRate - 22) * 0.03 - shortagePenalty * 0.35 - Math.max(0, -energyGap) * 0.08, 0, 100);
    company.hist.push(company.price);
    if(company.hist.length > 1040) company.hist.shift();
    avgGrowth += (company.price - company.prevPrice) / company.prevPrice;
  });
  avgGrowth /= Math.max(1, companies.length);
  avgRevenueChange /= Math.max(1, companies.length);
  progressCompanyRoadmaps();
  progressProductIndustrialization();
  progressForeignMarkets();

  S.gdp *= 1 + avgGrowth * 0.01 + S.businessCycle * 0.00045 + Math.max(0, energyGap) * 0.00035 - Math.max(0, -energyGap) * 0.0006;
  S.pop += (Math.random() - 0.507) * 0.2 + (S.educationBudget > 10 ? 0.045 : 0) - Math.max(0, S.unemployment - 7) * 0.01;

  S.approval +=
    (S.money - S.prevMoney > 0 ? 0.03 : -0.06) +
    S.educationBudget * 0.012 +
    S.infraLevel * 0.01 +
    workScale.hospital * 0.02 +
    Math.max(0, S.businessCycle) * 0.07 -
    Math.max(0, S.inflation - 4) * 0.045 -
    Math.max(0, S.unemployment - 6) * 0.03 -
    Math.max(0, S.taxRate - 20) * 0.03 -
    Math.max(0, -energyGap) * 0.08 -
    (S.activeMajorEvent ? 0.05 : 0);
  if(S.bubbleActive) S.approval -= 0.08;
  S.approval = clamp(S.approval, 0, 100);

  S.polCap = Math.min(S.polCapMax, S.polCap + 0.38 + S.approval * 0.004 + S.stability * 0.0012 + Math.max(0, S.businessCycle) * 0.18);

  const shortageCount = getShortageCount();
  const debtPressure = getTotalDebt() / Math.max(10000, S.gdp * 8);
  S.inflation = clamp(1.6 + Math.max(0, -S.businessCycle) * 0.5 + S.bubbleIndex * 0.05 + debtPressure * 9 + shortageCount * 0.13 - S.infraLevel * 0.09 - S.nuclearPlants * 0.04 + (100 / S.yenValue - 1) * 2.6 + Math.max(0, -energyGap) * 2.2 - Math.max(0, S.taxRate - 18) * 0.06 + rand(-0.22, 0.22), -1.2, 14);
  S.unemployment = clamp(5.4 - (S.gdp - 5400) / 620 - S.educationBudget * 0.055 - S.infraLevel * 0.06 - Math.max(0, S.businessCycle) * 1.1 + shortageCount * 0.18 + Math.max(0, -energyGap) * 0.9 + Math.max(0, S.taxRate - 24) * 0.05 + rand(-0.15, 0.15), 1.8, 16);
  S.stability = clamp(32 + S.approval * 0.44 + getAverageRelation() * 0.1 + S.defPower * 0.16 + S.cyberPower * 0.1 - S.bubbleIndex * 0.26 - Math.max(0, S.inflation - 4) * 2.6 - debtPressure * 40 - S.space.failures * 1.2 + Math.max(0, S.businessCycle) * 8 - Math.max(0, -energyGap) * 8, 0, 100);

  const speculativePressure = Math.max(0, companies.reduce((sum, company) => sum + Math.max(0, company.price / company.base - 1), 0) / companies.length);
  const subsidyHeatTotal = Object.values(S.subsidyHeat).reduce((sum, value) => sum + value, 0);
  const bubbleEnvironment =
    speculativePressure * 1.35
    + Math.max(0, S.businessCycle) * 0.85
    + Math.max(0, avgGrowth) * 42
    + Math.max(0, avgRevenueChange) * 30
    + subsidyHeatTotal * 0.02
    + (S.foreignUnrest > 16 ? 0.18 : 0)
    - Math.max(0, -S.businessCycle) * 1.6
    - Math.max(0, -avgGrowth) * 65
    - Math.max(0, -avgRevenueChange) * 40
    - S.socialUnrest * 0.018
    - Math.max(0, S.inflation - 4.8) * 0.16
    - (S.activeMajorEvent ? 0.34 : 0)
    - 0.14;
  S.bubbleIndex = clamp(S.bubbleIndex * 0.986 + bubbleEnvironment, 0, 320);
  if(!S.bubbleActive && (S.totalWeeks || 0) >= (S.nextBubbleEligibleWeek || 260) && (S.bubbleIndex > 52 || subsidyHeatTotal > 18)) triggerBubble();
  if(S.bubbleActive){
    const bubbleTension =
      speculativePressure * 0.75
      + Math.max(0, S.businessCycle) * 0.4
      + Math.max(0, avgGrowth) * 20
      - Math.max(0, -S.businessCycle) * 1.45
      - Math.max(0, -avgGrowth) * 52
      - S.socialUnrest * 0.02
      + rand(-0.6, 0.9);
    S.bubbleIndex = clamp(S.bubbleIndex + bubbleTension, 0, 320);
    if(S.bubbleIndex > 220 || bubbleTension < -1.15){
      S.bubbleActive = false;
      S.nextBubbleEligibleWeek = Math.max(S.nextBubbleEligibleWeek || 0, (S.totalWeeks || 0) + 260);
      companies.filter(company => company.sector === S.bubbleSector).forEach(company => company.price = Math.round(company.price * rand(0.42, 0.6)));
      companies.forEach(company => { if(company.sector !== S.bubbleSector) company.price = Math.round(company.price * rand(0.72, 0.92)); });
      S.gdp *= 0.92;
      S.approval = Math.max(0, S.approval - 10);
      S.bubbleIndex = 8;
      addEvent(`💥 バブル崩壊！${sectors[S.bubbleSector].name} が急落し、実体経済も大きく後退`, 'bad');
      S.bubbleSector = null;
    }
  }

  if(S.stability < 35){
    S.approval = Math.max(0, S.approval - 0.2);
    S.polCap = Math.max(0, S.polCap - 0.12);
    if(S.totalWeeks % 12 === 0) addEvent('🪧 社会不安が高まり政策遂行力が低下', 'bad');
  }
  if(S.inflation > 6.5 && S.totalWeeks % 14 === 0) addEvent('🛒 物価高騰が家計を圧迫', 'bad');
  if(S.unemployment > 7.5 && S.totalWeeks % 18 === 0) addEvent('🏢 雇用不安で消費心理が悪化', 'bad');
  if(S.space.factories > 0 && Math.random() < 0.014){
    S.space.resources += 2 + S.space.factories;
    S.money += 180 * S.space.factories;
    addEvent('🛰 軌道産業の受注拡大で宇宙部門が黒字化', 'good');
  }

  if(Math.random() < 0.018){
    const events = [
      {msg:'🌊 台風と高潮で沿岸部の供給網が停止', fn:()=>{ applyRegionalDisaster('台風高潮', pick(['kanto','kinki','kyushu','tohoku']), rand(0.35,0.6)); companies.filter(company => company.sector==='construction').forEach(company => company.price=Math.round(company.price*rand(1.02,1.08))); }},
      {msg:'🏭 半導体在庫調整で技術株が売られる', fn:()=>{ resources.semiconductor.amount *= 1.08; companies.filter(company => ['tech','telecom'].includes(company.sector)).forEach(company => company.price = Math.round(company.price * rand(0.9, 0.97))); }},
      {msg:'🤝 新たな国際貿易協定締結', fn:()=>{ tradePartners.forEach(partner => partner.relation = Math.min(100, partner.relation + rand(0.5, 2.5))); S.tradeDeals.forEach(deal=>{ if(!deal.isSell) deal.cost *= 0.94; else deal.cost *= 1.06; }); S.approval += 1.4; }},
      {msg:'🔬 画期的な技術ブレイクスルー', fn:()=>{ companies.filter(company => ['tech','space','hospital'].includes(company.sector)).forEach(company => company.price=Math.round(company.price*rand(1.02,1.06))); S.gdp*=1.0011; S.techLevel += 0.9; }},
      {msg:'🛒 生活コスト上昇への不満', fn:()=>{ if(S.inflation>5){ S.approval -= 2.8; S.stability = Math.max(0, S.stability - 2.2);} else {S.approval += 0.8;} }},
      {msg:'💹 海外投資家が短期的に日本株へ流入', fn:()=>{ companies.forEach(company => company.price = Math.round(company.price * rand(1.004,1.018))); S.foreignUnrest += 5; S.subsidyHeat.finance = (S.subsidyHeat.finance || 0) + 2; }},
    ];
    const ev = pick(events);
    ev.fn();
    addEvent(ev.msg, ev.msg.includes('協定') || ev.msg.includes('ブレイク') || ev.msg.includes('流入') ? 'good' : 'bad');
  }

  companies.forEach(company => {
    company.base = Math.max(100, Math.round(company.base));
    company.price = Math.max(100, Math.round(company.price));
    company.revenue = Math.min(company.revenue, getCompanyRevenueCeiling(company));
    company.revenue = Math.max(120, Math.round(company.revenue));
  });
  ensureForeignCompanies();
  S.foreignCompanies.forEach(company => {
    company.base = Math.max(100, Math.round(company.base));
    company.price = Math.max(100, Math.round(company.price));
    company.revenue = Math.min(company.revenue, Math.max(420, (company.baseRevenue || company.revenue || 420) * 5.2));
    company.revenue = Math.max(120, Math.round(company.revenue));
  });

  checkNationalGoals();
  pushHistoryPoint();
  maybeAutoSave();

  if(S.sandboxMode){
    S.money = Math.max(S.money, 250000);
    S.polCap = Math.max(S.polCap, 1800);
  } else if(S.approval <= 0){
    addEvent('☠ 支持率が 0% に達し、政権が崩壊した', 'bad');
    triggerGameOverSequence(S.language === 'en' ? 'Approval fell to zero. The cabinet collapsed and the nation has to restart.' : '支持率が 0% に達し、政権が崩壊しました。');
    return;
  }

  if(!options.skipUI){
    try{
      updateUI();
    } catch(error){
      reportRuntimeError('updateUI:tick', error);
    }
  }
}

// ============================================================
// UI UPDATE
// ============================================================
function updateUI(){
  const units = currentI18n().units || {};
  document.body.classList.toggle('focus-ui', !!S.focusMode);
  document.body.classList.toggle('mobile-ui', S.uiMode === 'mobile');
  document.body.classList.toggle('mobile-top-collapsed', S.uiMode === 'mobile' && !!S.mobileTopCollapsed);
  document.body.classList.toggle('mobile-bottom-collapsed', S.uiMode === 'mobile' && !!S.mobileBottomCollapsed);
  document.body.classList.toggle('mobile-panel-collapsed', S.uiMode === 'mobile' && !!S.mobilePanelCollapsed);
  document.body.classList.toggle('quality-low', S.qualityMode === 'low');
  document.body.classList.toggle('quality-medium', S.qualityMode === 'balanced');
  document.getElementById('money').textContent = formatMoneyCompact(S.money);
  document.getElementById('polCap').textContent = Math.round(S.polCap) + '/' + S.polCapMax;
  document.getElementById('pop').textContent = formatPopulationCompact(S.pop, 1);
  document.getElementById('gdp').textContent = formatGdpCompact(S.gdp, 1);
  document.getElementById('approval').textContent = S.approval.toFixed(1) + '%';
  document.getElementById('bubbleIdx').textContent = S.bubbleIndex.toFixed(0) + '%';
  document.getElementById('bubbleIdx').style.color = S.bubbleIndex>40?'#e94560':S.bubbleIndex>20?'#f39c12':'#4ecca3';
  document.getElementById('yenValue').textContent = S.yenValue.toFixed(1);
  document.getElementById('techLvl').textContent = S.techLevel.toFixed(1);
  document.getElementById('defPow').textContent = Math.round(S.defPower);
  document.getElementById('cyberPow').textContent = Math.round(S.cyberPower);
  document.getElementById('nukePow').textContent = S.nukePower;
  document.getElementById('sandboxState').textContent = S.sandboxMode ? 'ON' : 'OFF';
  document.getElementById('sandboxState').style.color = S.sandboxMode ? '#4ecca3' : '#aaa';
  document.getElementById('weekDisplay').textContent = formatWeekText();
  
  // Rates
  setRate('moneyRate', 0, '');
  setRate('polCapRate', S.polCap - S.prevPolCap, units.perWeek || '/wk');
  setRate('popRate', 0, '');
  setRate('gdpRate', 0, '');
  const moneyRateEl = document.getElementById('moneyRate');
  const popRateEl = document.getElementById('popRate');
  const gdpRateEl = document.getElementById('gdpRate');
  if(moneyRateEl){
    const moneyText = formatSignedCompact(getWeeklyMoneyNet(), amount => formatMoneyCompact(amount), units.perWeek || '/wk');
    moneyRateEl.textContent = moneyText;
    moneyRateEl.className = 'rate ' + (getWeeklyMoneyNet() >= 0 ? 'rate-pos' : 'rate-neg');
  }
  if(popRateEl){
    const popText = formatSignedCompact(S.pop - S.prevPop, amount => formatPopulationCompact(amount, 1), units.perWeek || '/wk');
    popRateEl.textContent = popText;
    popRateEl.className = 'rate ' + ((S.pop - S.prevPop) >= 0 ? 'rate-pos' : 'rate-neg');
  }
  if(gdpRateEl){
    const gdpText = formatSignedCompact(S.gdp - S.prevGdp, amount => formatGdpCompact(amount, 1), units.perWeek || '/wk');
    gdpRateEl.textContent = gdpText;
    gdpRateEl.className = 'rate ' + ((S.gdp - S.prevGdp) >= 0 ? 'rate-pos' : 'rate-neg');
  }
  setRate('approvalRate', S.approval - S.prevApproval, `%${units.perWeek || '/wk'}`);
  setRate('yenRate', S.yenValue - S.prevYenValue, units.perWeek || '/wk');
  setRate('techRate', S.techLevel - S.prevTechLevel, units.perWeek || '/wk');
  
  updateSpeedButtons();
  updatePriceBar();
  schedulePanelRender();
  scheduleMapDraw();
  scheduleMapHudRefresh();
  renderFocusTray();
  renderFocusTimeDock();
  renderHistoryDock();
  initFloatingDocks();
  initMapHudDrag();
  syncFloatingDockState();
  const bottomFocusBtn = document.getElementById('bottomFocusBtn');
  const mapFocusModeBtn = document.getElementById('mapFocusModeBtn');
  if(bottomFocusBtn) bottomFocusBtn.classList.toggle('active', !!S.focusMode);
  if(mapFocusModeBtn) mapFocusModeBtn.classList.toggle('active', !!S.focusMode);
  const topToggle = document.getElementById('mobileTopToggle');
  const bottomToggle = document.getElementById('mobileBottomToggle');
  const panelToggle = document.getElementById('mobilePanelToggle');
  if(topToggle){
    topToggle.style.display = S.uiMode === 'mobile' ? 'flex' : 'none';
    topToggle.textContent = S.mobileTopCollapsed ? '▾' : '▴';
    topToggle.title = S.language === 'en' ? (S.mobileTopCollapsed ? 'Open top bar' : 'Collapse top bar') : (S.mobileTopCollapsed ? '上バーを開く' : '上バーをたたむ');
  }
  if(bottomToggle){
    bottomToggle.style.display = S.uiMode === 'mobile' ? 'flex' : 'none';
    bottomToggle.textContent = S.mobileBottomCollapsed ? '▴' : '▾';
    bottomToggle.title = S.language === 'en' ? (S.mobileBottomCollapsed ? 'Open bottom bar' : 'Collapse bottom bar') : (S.mobileBottomCollapsed ? '下バーを開く' : '下バーをたたむ');
  }
  if(panelToggle){
    panelToggle.style.display = S.uiMode === 'mobile' ? 'flex' : 'none';
    panelToggle.textContent = S.mobilePanelCollapsed ? '◀' : '▶';
    panelToggle.title = S.language === 'en' ? (S.mobilePanelCollapsed ? 'Open right panel' : 'Collapse right panel') : (S.mobilePanelCollapsed ? '右パネルを開く' : '右パネルをたたむ');
  }
  const mapHudFoldBtn = document.getElementById('mapHudFoldBtn');
  if(mapHudFoldBtn){
    mapHudFoldBtn.style.display = 'inline-flex';
  }
  const actionModal = document.getElementById('actionModal');
  if(actionModal?.classList.contains('show') && actionModal.dataset.action === 'top-stat' && actionModal.dataset.topStat){
    openTopStatDetail(actionModal.dataset.topStat);
  }
  maybePerformPeriodicUiMaintenance();
}

function setRate(id, val, suffix){
  const el = document.getElementById(id);
  const abs = Math.abs(val);
  const fmt = abs >= 100 ? val.toFixed(0) : abs >= 1 ? val.toFixed(1) : val.toFixed(2);
  el.textContent = (val >= 0 ? '+' : '') + fmt + suffix;
  el.className = 'rate ' + (val >= 0 ? 'rate-pos' : 'rate-neg');
}

function updateSpeedButtons(){
  const speedButtons = document.querySelectorAll('#speedCtrl [data-speed]');
  speedButtons.forEach(button => {
    const speed = Number(button.dataset.speed || 0);
    button.classList.toggle('active', S.timeMode === 'realtime' && S.speed === speed);
    button.disabled = S.timeMode !== 'realtime';
  });
  const realtimeBtn = document.getElementById('timeRealtimeBtn');
  const turnBtn = document.getElementById('timeTurnBtn');
  const turnStepBtn = document.getElementById('turnStepBtn');
  const advanceBtn = document.getElementById('turnAdvanceBtn');
  if(realtimeBtn) realtimeBtn.classList.toggle('active', S.timeMode === 'realtime');
  if(turnBtn) turnBtn.classList.toggle('active', S.timeMode === 'turn');
  if(turnStepBtn){
    turnStepBtn.textContent = getTurnStepLabel(S.turnStepWeeks);
    turnStepBtn.style.display = S.timeMode === 'turn' ? '' : 'none';
    turnStepBtn.disabled = S.turnProcessing;
  }
  if(advanceBtn){
    advanceBtn.style.display = S.timeMode === 'turn' ? '' : 'none';
    advanceBtn.disabled = S.turnProcessing || !S.started;
  }
  renderFocusTimeDock();
}

function setSpeed(s){
  if(S.timeMode === 'turn'){
    if(s <= 0){
      updateSpeedButtons();
      return;
    }
    S.timeMode = 'realtime';
  }
  S.speed = s;
  if(s > 0) S.realtimeSpeed = s;
  updateSpeedButtons();
}

// ============================================================
// MAIN LOOP
// ============================================================
let lastTick = 0;
const BASE_TICK = 5000;
let lastAnimatedMapFrame = 0;

function gameLoop(ts){
  if(S.timeMode === 'realtime' && S.speed > 0 && ts - lastTick >= BASE_TICK / Math.max(1, S.speed)){
    lastTick = ts;
    try{
      gameTick();
    } catch(error){
      console.error('gameTick error', error);
      reportRuntimeError('gameTick', error);
      addEvent('⚠ 実行エラーを検知したため時間を停止しました', 'bad');
      setSpeed(0);
    }
  }
  try{
    const quality = getQualityProfile();
    if(typeof hasAnimatedMapActivity === 'function' && hasAnimatedMapActivity() && ts - lastAnimatedMapFrame >= quality.animatedFrameMs){
      lastAnimatedMapFrame = ts;
      drawMap();
    }
  } catch(error){
    console.error('drawMap animation error', error);
    reportRuntimeError('gameLoop:drawMap', error);
  }
  requestAnimationFrame(gameLoop);
}

// ============================================================
// INIT
// ============================================================
window.addEventListener('keydown', e=>{
  if(e.target && ['INPUT','TEXTAREA'].includes(e.target.tagName)) return;
  if(e.code === 'Space'){
    e.preventDefault();
    if(S.timeMode === 'turn') advanceTurn();
    else setSpeed(S.speed === 0 ? Math.max(1, S.realtimeSpeed || 1) : 0);
  }
  if(e.key === '1') setSpeed(1);
  if(e.key === '2') setSpeed(2);
  if(e.key === '3') setSpeed(4);
});

window.addEventListener('beforeunload', ()=> {
  try{ autoSaveForRelog(); }catch(error){}
});

window.addEventListener('resize', ()=> {
  try{ resizeCanvas(); }catch(error){ reportRuntimeError('resizeCanvas', error); }
});
try{
  S.language = 'ja';
  persistLanguagePreference('ja');
  S.uiMode = 'desktop';
  S.mobileTopCollapsed = false;
  S.mobileBottomCollapsed = false;
  S.mobilePanelCollapsed = false;
  S.mobileMapHudCollapsed = false;
  ensureStateShape();
  finalizeStaticSecurity();
  seedInitialPublicWorks();
  normalizeSeededPublicWorks();
  pushHistoryPoint(true);
  loadViewForMode(S.currentMapMode);
  setMapFocus('panel', {skipResize:true});
  resizeCanvas();
  refreshIntegrityBaseline(true);
  auditEconomyIntegrity('boot');
  updateUI();
  captureInitialSnapshot();
  addEvent('🇯🇵 日本経済シミュレーター v4 開始','good');
  addEvent('💡 貿易で不足資源を確保し、開発で産出量を増やそう','');
  addEvent('💻 サイバー力を高めて貿易を有利に','');
  addEvent('☢ 原子力でエネルギー産業を発展させよう','');
  if(getSaveMeta('autosave')) addEvent('💾 セーブ管理からオートセーブを復元できます','');
  openHomeScreen('ホーム画面から開始できます。');
}catch(error){
  reportRuntimeError('boot', error);
}
// ============================================================
// ENHANCEMENT LAYER - 2026 APRIL PASS
// ============================================================
(function(){
  return;
  const SAVE_SECRET = 'jp-econ-sim-v4-2026-secure';
  const rawGetSaveMeta = typeof getSaveMeta === 'function' ? getSaveMeta : null;
  const rawPerformSave = typeof performSave === 'function' ? performSave : null;
  const rawLoadGame = typeof window.loadGame === 'function' ? window.loadGame : null;
  const rawBuyStock = typeof window.buyStock === 'function' ? window.buyStock : null;
  const rawUpdateUI = typeof updateUI === 'function' ? updateUI : function(){};
  const rawRenderSpace = typeof renderSpace === 'function' ? renderSpace : function(){ return ''; };
  const rawLaunchSpaceMission = typeof window.launchSpaceMission === 'function' ? window.launchSpaceMission : null;
  const rawResolveSpaceMission = typeof resolveSpaceMission === 'function' ? resolveSpaceMission : null;
  const rawGameTick = typeof gameTick === 'function' ? gameTick : function(){};
  const rawOpenCompanyDetail = typeof window.openCompanyDetail === 'function' ? window.openCompanyDetail : null;
  const rawDrawMap = typeof drawMap === 'function' ? drawMap : function(){};
  const rawCanPlacePublicWork = typeof canPlacePublicWork === 'function' ? canPlacePublicWork : null;
  const rawRenderPanel = typeof renderPanel === 'function' ? renderPanel : function(){};
  const rawCloseHomeScreen = typeof closeHomeScreen === 'function' ? closeHomeScreen : null;
  const moneyCompact = typeof formatMoneyCompact === 'function'
    ? formatMoneyCompact
    : function(value){
        const abs = Math.abs(value);
        if(abs >= 100000000) return (value / 100000000).toFixed(2) + '兆';
        if(abs >= 10000) return (value / 10000).toFixed(1) + '億';
        return Math.round(value).toLocaleString();
      };
  const amountCompact = typeof formatAmount === 'function'
    ? formatAmount
    : function(value){
        if(value >= 1000000) return (value / 1000000).toFixed(1) + 'M';
        if(value >= 1000) return (value / 1000).toFixed(1) + 'K';
        return value.toFixed(0);
      };

  function xorString(value, key){
    let out = '';
    for(let i = 0; i < value.length; i++) out += String.fromCharCode(value.charCodeAt(i) ^ key.charCodeAt(i % key.length));
    return out;
  }

  function encodeSavePayload(payload){
    try{
      const json = JSON.stringify(payload);
      return 'ENC1:' + btoa(xorString(json, SAVE_SECRET));
    }catch(error){
      return JSON.stringify(payload);
    }
  }

  function decodeSavePayload(raw){
    if(!raw) return null;
    if(raw.startsWith('ENC1:')){
      const decoded = atob(raw.slice(5));
      return JSON.parse(xorString(decoded, SAVE_SECRET));
    }
    return JSON.parse(raw);
  }

  function ensureCompanyModels(){
    companies.forEach((company, index) => {
      if(typeof company.sharesOutstanding !== 'number') company.sharesOutstanding = 2000 + index * 55 + Math.round((company.base || company.price || 1000) / 45);
      if(typeof company.baseRevenue !== 'number') company.baseRevenue = Math.max(600, Math.round((company.base || company.price || 1000) * (1.2 + (index % 5) * 0.18)));
      if(typeof company.revenue !== 'number') company.revenue = company.baseRevenue;
      if(!Array.isArray(company.revenueHist)) company.revenueHist = [company.revenue];
      if(typeof company.baseDemand !== 'number') company.baseDemand = rand(92, 108);
      if(typeof company.demand !== 'number') company.demand = company.baseDemand;
      if(typeof company.prevDemand !== 'number') company.prevDemand = company.demand;
      if(!Array.isArray(company.demandHist)) company.demandHist = [company.demand];
      if(typeof company.techLevel !== 'number') company.techLevel = clamp(8 + (company.sector === 'tech' ? 18 : 0) + (company.sector === 'space' ? 24 : 0) + (company.sector === 'pharma' ? 12 : 0) + (company.sector === 'defense' ? 8 : 0), 5, 95);
      if(typeof company.ownerEvent !== 'boolean') company.ownerEvent = false;
      if(!Array.isArray(company.sites)) company.sites = [];
      if(typeof company.workers !== 'number') company.workers = 180 + index * 9;
      if(typeof company.dividendRate !== 'number') company.dividendRate = 0.0016;
      if(company.price < 100) company.price = 100;
      if(!Array.isArray(company.hist)) company.hist = [company.price];
      ensureRevenueWaveState(company);
    });
    if(typeof S.techLevel !== 'number') S.techLevel = 16;
    if(typeof S.policyHeat !== 'number') S.policyHeat = 0;
    if(typeof S.marketShock !== 'number') S.marketShock = 0;
    if(typeof S.lastPowerAlertWeek !== 'number') S.lastPowerAlertWeek = -999;
    if(!Array.isArray(S.majorEvents)) S.majorEvents = [];
    if(!S.energy) S.energy = {production:0, demand:0};
    if(!S.space) S.space = createSpaceState();
    if(typeof S.space.rocketLevel !== 'number') S.space.rocketLevel = 0;
    if(typeof S.space.solarUnlocked !== 'boolean') S.space.solarUnlocked = false;
    if(!S.space.planets || !Object.keys(S.space.planets).length) S.space.planets = createPlanetResourceState();
  if(!S.weeklyBreakdown) S.weeklyBreakdown = {tax:0,budget:0,trade:0,dividend:0,space:0,interest:0,principal:0,companySupport:0,publicRevenue:0,operations:0,socialSecurity:0,healthcare:0,childcare:0,administration:0,adjustments:0};
    if(typeof S.lastOwnershipEventWeek !== 'number') S.lastOwnershipEventWeek = -999;
    if(!Array.isArray(S.companyFacilities)) S.companyFacilities = [];
    if(!S.draggingFacility) S.draggingFacility = null;
    if(!S.foreignOwnedStocks || typeof S.foreignOwnedStocks !== 'object') S.foreignOwnedStocks = {};
    if(!Array.isArray(S.worldLog)) S.worldLog = [];
    if(typeof S.foreignDividend !== 'number') S.foreignDividend = 0;
    if(!Array.isArray(S.foreignCompanies)) S.foreignCompanies = [];
    if(typeof S.loopStarted !== 'boolean') S.loopStarted = false;
  }

  function ownershipRatio(company){
    ensureCompanyModels();
    return clamp(((S.ownedStocks[company.name] || 0) / Math.max(1, company.sharesOutstanding)) * 100, 0, 100);
  }

  function getPlanetState(bodyId){
    ensureCompanyModels();
    if(!S.space.planets[bodyId]) S.space.planets = createPlanetResourceState();
    return S.space.planets[bodyId] || {total:{}, remaining:{}, developed:false};
  }

  function getSpaceCompanies(){
    return companies.filter(company => ['space','tech','telecom','defense'].includes(company.sector));
  }

  function getPlanetMaterials(body){
    const materials = [];
    (body.materials || []).forEach(material => materials.push(material));
    (body.rewards || []).forEach(material => {
      if(!materials.find(existing => existing.id === material.id)) materials.push(material);
    });
    return materials;
  }

  function buildForeignCompanies(){
    const templates = [
      {name:'Aegis Dynamics', partner:0, sector:'defense', basePrice:1480, baseRevenue:1900},
      {name:'Sino Materials', partner:1, sector:'mining', basePrice:920, baseRevenue:2100},
      {name:'Outback Energy', partner:2, sector:'energy', basePrice:860, baseRevenue:1750},
      {name:'Desert Petro', partner:3, sector:'energy', basePrice:1100, baseRevenue:2280},
      {name:'Rhine Systems', partner:4, sector:'tech', basePrice:1760, baseRevenue:2020},
      {name:'Andes Lithium', partner:6, sector:'mining', basePrice:980, baseRevenue:1690},
      {name:'Pacific Meditech', partner:8, sector:'hospital', basePrice:1320, baseRevenue:1580},
      {name:'Bharat Infra', partner:9, sector:'construction', basePrice:1020, baseRevenue:1880},
    ];
    return templates.map((template, index) => {
      const partner = tradePartners[template.partner];
      return {
        id:`fc-${index}`,
        name:template.name,
        partnerIndex:template.partner,
        country:partner?.name || 'Unknown',
        sector:template.sector,
        price:template.basePrice,
        prevPrice:template.basePrice,
        base:template.basePrice,
        baseRevenue:template.baseRevenue,
        revenue:template.baseRevenue,
        revenueHist:[template.baseRevenue],
        hist:[template.basePrice],
        sharesOutstanding:3200 + index * 440,
        techLevel:18 + index * 3,
        margin:0.1 + index * 0.004
      };
    });
  }

  function ensureForeignCompanies(){
    ensureCompanyModels();
    if(!S.foreignCompanies.length) S.foreignCompanies = buildForeignCompanies();
    S.foreignCompanies.forEach((company, index) => {
      if(typeof company.sharesOutstanding !== 'number') company.sharesOutstanding = 3200 + index * 440;
      if(typeof company.baseRevenue !== 'number') company.baseRevenue = company.revenue || 1800;
      if(typeof company.revenue !== 'number') company.revenue = company.baseRevenue;
      if(!Array.isArray(company.revenueHist)) company.revenueHist = [company.revenue];
      if(!Array.isArray(company.hist)) company.hist = [company.price];
      if(typeof company.margin !== 'number') company.margin = 0.12;
      if(company.price < 100) company.price = 100;
    });
  }

  function foreignOwnershipRatio(company){
    return clamp(((S.foreignOwnedStocks[company.id] || 0) / Math.max(1, company.sharesOutstanding)) * 100, 0, 100);
  }

  function ensurePlanetGlobalResource(material, body){
    if(resources[material.id]) return resources[material.id];
    resources[material.id] = {
      name: material.name || material.id,
      amount: 0,
      max: Math.max(1000, material.total || 1000),
      color: material.color || '#9ad1ff',
      domestic: false,
      region: body ? body.id : 'space',
      price: material.price || material.basePrice || 150,
      priceHist: [material.price || material.basePrice || 150],
      unit: material.unit || 't',
      isSpace:true
    };
    return resources[material.id];
  }

  function getTravelWeeksForBody(body){
    const base = typeof body.travelWeeks === 'number'
      ? body.travelWeeks
      : body.id === 'moon'
        ? 18
        : body.id === 'mars'
          ? 34
          : body.id === 'asteroid'
            ? 42
            : body.id === 'europa'
              ? 58
              : body.id === 'titan'
                ? 74
                : 96;
    return Math.max(10, Math.round(base * Math.max(0.45, 1.28 - S.space.rocketLevel * 0.06)));
  }

  function getRocketRequirement(body){
    if(typeof body.rocketRequirement === 'number') return body.rocketRequirement;
    if(body.id === 'moon') return 1;
    if(body.id === 'mars') return 2;
    if(body.id === 'asteroid') return 3;
    if(body.id === 'europa') return 4;
    if(body.id === 'titan') return 5;
    return 6;
  }

  function getRocketDevelopmentCost(nextLevel){
    return 14000 + nextLevel * 11000;
  }

  function getTradeFlowValue(){
    return S.tradeDeals.reduce((sum, deal) => sum + (deal.isSell ? Math.abs(deal.cost) : -Math.abs(deal.cost)), 0);
  }

  function getBudgetOutflowWeekly(){
    ensureCompanyModels();
    return (S.annualBudget.military + S.annualBudget.education + S.annualBudget.technology + S.annualBudget.companySupport) / 52;
  }

  function repairCoreUI(){
    let rightPanel = document.getElementById('rightPanel');
    const gameAreaEl = document.getElementById('gameArea');
    if(!rightPanel){
      rightPanel = document.createElement('div');
      rightPanel.id = 'rightPanel';
      const bottomBar = document.getElementById('bottomBar');
      document.body.insertBefore(rightPanel, bottomBar || null);
    }
    let panelTabs = document.getElementById('panelTabs');
    if(!panelTabs){
      panelTabs = document.createElement('div');
      panelTabs.id = 'panelTabs';
      rightPanel.appendChild(panelTabs);
    }
    let panelContent = document.getElementById('panelContent');
    if(rightPanel){
      rightPanel.style.display = 'flex';
      rightPanel.style.visibility = 'visible';
      rightPanel.style.opacity = '1';
    }
    if(gameAreaEl){
      gameAreaEl.style.display = 'block';
      gameAreaEl.style.visibility = 'visible';
      gameAreaEl.style.opacity = '1';
    }
    if(panelTabs){
      panelTabs.innerHTML = buildPanelTabsHtml();
      if(!panelTabs.dataset.bound){
        panelTabs.dataset.bound = '1';
        panelTabs.addEventListener('click', event => {
          const button = event.target.closest('[data-panel]');
          if(!button) return;
          switchPanelView(button.dataset.panel, {useLoader:false});
        });
      }
    }
    if(rightPanel && !panelContent){
      panelContent = document.createElement('div');
      panelContent.id = 'panelContent';
      rightPanel.appendChild(panelContent);
    }
    if(panelContent){
      panelContent.style.display = 'block';
      panelContent.style.visibility = 'visible';
    }
    if(typeof loadViewForMode === 'function') loadViewForMode(S.currentMapMode || 'japan');
    if(typeof resizeCanvas === 'function') resizeCanvas();
    if(typeof renderPanel === 'function') renderPanel();
    if(typeof drawMap === 'function') drawMap();
    if(typeof updateChromeLanguage === 'function') updateChromeLanguage();
  }

  canPlacePublicWork = function(workId, worldPos){
    const baseVerdict = rawCanPlacePublicWork ? rawCanPlacePublicWork(workId, worldPos) : {ok:true, regionKey:getClosestRegion(worldPos.x, worldPos.y)};
    if(!baseVerdict.ok) return baseVerdict;
    const regionKey = baseVerdict.regionKey;
    const coastalRegions = ['kanto','chubu','kinki','chugoku','kyushu','okinawa'];
    const solarRegions = ['tohoku','kanto','chubu','kinki','kyushu','okinawa'];
    if(workId === 'thermal' && !coastalRegions.includes(regionKey)) return {ok:false, regionKey, reason:'火力発電所は沿岸工業地帯に近い地域へ配置してください'};
    if(workId === 'solar' && !solarRegions.includes(regionKey)) return {ok:false, regionKey, reason:'太陽光施設は日照の取りやすい地域を選んでください'};
    if(workId === 'hydro' && !isNearRiver(worldPos.x, worldPos.y, 0.03)) return {ok:false, regionKey, reason:'ダムは川の上流や大河川沿いにのみ配置できます'};
    return {ok:true, regionKey};
  };

  function beginCompanyFacilityPlacement(companyName, facilityType){
    const company = companies.find(item => item.name === companyName);
    if(!company) return;
    if(ownershipRatio(company) < 100){
      addEvent('❌ 100%保有企業のみ施設を配置できます', 'bad');
      return;
    }
    const config = {
      factory:{label:'工場', cost:6200, workers:160, color:'#90caf9'},
      hospital:{label:'病院', cost:5400, workers:130, color:'#ef9a9a'},
      lab:{label:'研究所', cost:7600, workers:90, color:'#ce93d8'}
    }[facilityType];
    if(!config) return;
    if(S.money < config.cost){ addEvent('❌ 施設建設資金が足りません', 'bad'); return; }
    S.draggingFacility = {
      companyName,
      type:facilityType,
      label:config.label,
      cost:config.cost,
      workers:config.workers,
      color:config.color,
      x:regions[company.region]?.cx || regions.kanto.cx,
      y:regions[company.region]?.cy || regions.kanto.cy
    };
    S.currentMapMode = 'japan';
    loadViewForMode('japan');
    drawMap();
    addEvent(`🏭 ${companyName} の${config.label}配置モード開始。日本地図をクリックしてください`, 'good');
  }

  function canPlaceCompanyFacility(payload, worldPos){
    const regionKey = getClosestRegion(worldPos.x, worldPos.y);
    if(!regionKey) return {ok:false, reason:'日本地図の陸地に配置してください'};
    if(payload.type === 'hospital' && ['okinawa','sea_pacific','sea_japan','sea_east'].includes(regionKey)) return {ok:false, reason:'病院は主要陸地に配置してください'};
    if(payload.type === 'factory' && ['tohoku','kanto','chubu','kinki','kyushu','chugoku'].indexOf(regionKey) === -1) return {ok:false, reason:'工場は産業集積地に配置してください'};
    if(payload.type === 'lab' && ['kanto','chubu','kinki'].indexOf(regionKey) === -1) return {ok:false, reason:'研究所は大都市圏に配置してください'};
    return {ok:true, regionKey};
  }

  function placeCompanyFacility(payload, worldPos){
    const verdict = canPlaceCompanyFacility(payload, worldPos);
    if(!verdict.ok){
      addEvent(`❌ ${verdict.reason}`, 'bad');
      return;
    }
    const company = companies.find(item => item.name === payload.companyName);
    if(!company) return;
    S.money -= payload.cost;
    const facility = {
      id:`cf-${Date.now()}-${Math.random().toString(36).slice(2,6)}`,
      companyName:payload.companyName,
      type:payload.type,
      label:payload.label,
      x:worldPos.x,
      y:worldPos.y,
      region:verdict.regionKey,
      workers:payload.workers,
      damagedWeeks:0
    };
    S.companyFacilities.push(facility);
    company.sites.push(facility.id);
    company.workers += payload.workers;
    company.revenue += payload.cost * 0.18;
    company.price = Math.max(100, Math.round(company.price * 1.012));
    addEvent(`🏗 ${payload.companyName} が ${regions[verdict.regionKey].name} に${payload.label}を建設`, 'good');
  }

  function adjustCompanyWorkers(companyName, delta){
    const company = companies.find(item => item.name === companyName);
    if(!company || ownershipRatio(company) < 100) return;
    company.workers = Math.max(40, company.workers + delta);
    company.revenue = Math.max(120, company.revenue + delta * 0.9);
    company.techLevel = clamp(company.techLevel + delta / 900, 0, 100);
    addEvent(`👷 ${companyName} の作業員を ${delta > 0 ? '+' : ''}${delta} 調整`, 'good');
    openCompanyDetail(companyName);
  }

  function buyForeignStock(companyId, qty){
    ensureForeignCompanies();
    const company = S.foreignCompanies.find(item => item.id === companyId);
    if(!company) return;
    const owned = S.foreignOwnedStocks[companyId] || 0;
    const remaining = Math.max(0, company.sharesOutstanding - owned);
    if(remaining <= 0){ addEvent('❌ これ以上は購入できません', 'bad'); return; }
    qty = Math.min(qty, remaining);
    const cost = company.price * qty;
    if(S.money < cost){ addEvent('❌ 資金不足', 'bad'); return; }
    S.money -= cost;
    S.foreignOwnedStocks[companyId] = owned + qty;
    addEvent(`🌍 ${company.name} を ${qty}株購入`, 'good');
    openForeignCompanyDetail(companyId);
  }

  function sellForeignStock(companyId, qty){
    ensureForeignCompanies();
    const company = S.foreignCompanies.find(item => item.id === companyId);
    if(!company) return;
    const owned = S.foreignOwnedStocks[companyId] || 0;
    qty = Math.min(qty, owned);
    if(qty <= 0) return;
    S.money += company.price * qty;
    S.foreignOwnedStocks[companyId] = owned - qty;
    if(S.foreignOwnedStocks[companyId] <= 0) delete S.foreignOwnedStocks[companyId];
    addEvent(`🌍 ${company.name} を ${qty}株売却`, 'good');
    openForeignCompanyDetail(companyId);
  }

  window.openForeignCompanyDetail = function(companyId){
    ensureForeignCompanies();
    const company = S.foreignCompanies.find(item => item.id === companyId);
    if(!company) return;
    const owned = S.foreignOwnedStocks[companyId] || 0;
    const ratio = foreignOwnershipRatio(company);
    document.getElementById('modalTitle').textContent = `🌍 ${company.name}`;
    document.getElementById('modalContent').innerHTML = `<div class="detail-grid">
      <div class="dg-item"><div class="dg-label">国</div><div class="dg-value">${company.country}</div></div>
      <div class="dg-item"><div class="dg-label">業種</div><div class="dg-value">${sectors[company.sector]?.name || company.sector}</div></div>
      <div class="dg-item"><div class="dg-label">株価</div><div class="dg-value">${moneyCompact(company.price)}</div></div>
      <div class="dg-item"><div class="dg-label">保有率</div><div class="dg-value">${ratio.toFixed(2)}%</div></div>
      <div class="dg-item"><div class="dg-label">売上</div><div class="dg-value">${moneyCompact(company.revenue)}</div></div>
      <div class="dg-item"><div class="dg-label">技術力</div><div class="dg-value">${company.techLevel.toFixed(1)}</div></div>
    </div>
    <div class="range-group">
      <button class="btn btn-green" onclick="buyForeignStock('${company.id}', 1)">1株</button>
      <button class="btn btn-green" onclick="buyForeignStock('${company.id}', 10)">10株</button>
      <button class="btn btn-green" onclick="buyForeignStock('${company.id}', 100)">100株</button>
      <button class="btn" onclick="sellForeignStock('${company.id}', 1)" ${owned<1?'disabled':''}>1株売却</button>
      <button class="btn" onclick="sellForeignStock('${company.id}', ${owned})" ${owned<1?'disabled':''}>全売却</button>
    </div>`;
    document.getElementById('actionModal').classList.add('show');
  };

  window.buyForeignStock = buyForeignStock;
  window.sellForeignStock = sellForeignStock;

  function renderWorld(){
    ensureForeignCompanies();
    const japanIntel = getDisplayMilitaryIntel('japan');
    const japanSnapshot = getWorldCountrySnapshot(getWorldCountry('japan'));
    const powers = tradePartners.map((partner, index) => {
      const companyCount = S.foreignCompanies.filter(company => company.partnerIndex === index).length;
      return `<div class="goal-card">
        <div class="goal-title"><span>${partner.name}</span><span>友好 ${Math.round(partner.relation)}</span></div>
        <div class="goal-desc">経済 ${partner.economy} / サイバー ${partner.cyberPower} / 核 ${partner.nukePower} / 企業 ${companyCount}社</div>
        <div class="compact-note">AI行動: ${partner.relation < 40 ? '強硬' : partner.relation > 70 ? '協調' : '警戒'} | 日本への圧力や投資が変動します。</div>
      </div>`;
    }).join('');
    const foreignMarket = S.foreignCompanies.map(company => {
      const owned = S.foreignOwnedStocks[company.id] || 0;
      return `<div class="company" style="border-left-color:${sectors[company.sector]?.color || '#888'}" onclick="openForeignCompanyDetail('${company.id}')">
        <div class="c-left"><span class="c-name">🌍${company.name}${owned>0 ? ' 🔷' + owned : ''}</span></div>
        <div class="c-right"><span class="c-change">${company.country}</span><span class="c-price">${moneyCompact(company.price)}</span></div>
      </div>`;
    }).join('');
    const worldLog = (S.worldLog || []).slice(0, 8).map(item => `<div class="advisor-item">${item}</div>`).join('') || '<div class="compact-note">まだ世界AIのログはありません。</div>';
    return `<div class="section"><h3>🌐 世界情勢</h3>
      <div class="headline-card good">
        <div class="meta">日本の戦力</div>
        <div class="body">GDP ${Math.round(japanSnapshot.gdp)}兆 / 防御戦力 ${Math.round(japanIntel?.power || 0)} / 歩 ${S.militaryInventory.infantry} / 戦 ${S.militaryInventory.tanks} / 航 ${S.militaryInventory.fighters} / 艦 ${S.militaryInventory.ships} / 無 ${S.militaryInventory.drones}</div>
      </div>${powers}</div>
      <div class="section"><h3>🌍 外国株市場</h3>${foreignMarket}</div>
      <div class="section"><h3>🤖 世界AIログ</h3>${worldLog}</div>`;
  }

  function getCompanyResourceScore(company){
    if(!company.needs || !company.needs.length) return 1;
    let total = 0;
    company.needs.forEach(key => {
      const resource = resources[key];
      if(!resource || !resource.max){
        total += 0.92;
        return;
      }
      const ratio = clamp(resource.amount / Math.max(1, resource.max), 0, 1.8);
      total += ratio <= 0.02 ? 0.62 : ratio <= 0.08 ? 0.78 : ratio <= 0.2 ? 0.9 : ratio >= 1 ? 1.06 : 0.97 + ratio * 0.08;
    });
    return total / company.needs.length;
  }

  function getRecentEventShock(company){
    const headline = S.eventHistory && S.eventHistory[0] ? S.eventHistory[0].msg : '';
    if(!headline) return 1;
    let shock = 1;
    if(/金融危機|景気後退|信用収縮/.test(headline)){
      if(['finance','retail','auto','tech'].includes(company.sector)) shock *= 0.72;
      if(company.sector === 'construction') shock *= 0.86;
    }
    if(/パンデミック|感染症|菌/.test(headline)){
      if(['retail','shipping','food','auto'].includes(company.sector)) shock *= 0.7;
      if(['pharma','hospital'].includes(company.sector)) shock *= 1.18;
    }
    if(/地震|津波/.test(headline)){
      if(company.sector === 'construction') shock *= 1.16;
      if(company.region && headline.includes(regions[company.region]?.name || '')) shock *= 0.64;
    }
    if(/原油|エネルギー危機/.test(headline)){
      if(['shipping','retail','auto'].includes(company.sector)) shock *= 0.8;
      if(company.sector === 'energy') shock *= 1.09;
    }
    if(/AI|技術ブレイクスルー/.test(headline) && ['tech','telecom','space'].includes(company.sector)) shock *= 1.1;
    return shock;
  }

  function smoothCompanyEconomics(){
    ensureCompanyModels();
    const difficulty = getDifficultyMeta();
    const fx = typeof S.yenValue === 'number' ? S.yenValue : 100;
    const fxExportBoost = fx < 100 ? 1 + (100 - fx) / 240 : 1 - (fx - 100) / 500;
    const fxImportPenalty = fx > 100 ? 1 - (fx - 100) / 280 : 1 + (100 - fx) / 420;
    const cycle = (Math.sin(S.totalWeeks / 10) * 0.06 + Math.sin(S.totalWeeks / 27) * 0.08 - S.policyHeat * 0.035 - S.marketShock * 0.15) * difficulty.marketSwing;
    const startupRatio = Math.max(0, 78 - S.totalWeeks) / 78;
    companies.forEach(company => {
      const facilityCount = S.companyFacilities.filter(item => item.companyName === company.name && !(item.damagedWeeks > 0)).length;
      const workerFactor = 1 + Math.max(0, company.workers - 180) / 5000;
      const resourceScore = getCompanyResourceScore(company);
      const macroScore = clamp(0.86 + (S.gdp / 5400 - 1) * 0.22 + (S.stability - 60) / 260 - Math.max(0, S.inflation - 2.2) / 24 - Math.max(0, S.unemployment - 4.5) / 36, 0.48, 1.55);
      const baseEventShock = getRecentEventShock(company);
      const eventShock = clamp(1 + (baseEventShock - 1) * difficulty.eventShockImpact, 0.42, 1.62);
      const exportFactor = ['auto','tech','shipping','space','defense'].includes(company.sector) ? fxExportBoost : 1;
      const importFactor = ['retail','food','energy'].includes(company.sector) ? fxImportPenalty : 1;
      const energyFactor = S.energy.production + 1 < S.energy.demand
        ? (['tech','auto','retail','space'].includes(company.sector) ? 0.84 : 0.92)
        : 1.02;
      const techFactor = clamp(0.92 + (company.techLevel / 100) * 0.26 + (S.techLevel / 100) * 0.08, 0.9, 1.42);
      const ownershipFactor = ownershipRatio(company) >= 100 ? 1.05 : 1;
      const siteFactor = 1 + facilityCount * 0.055;
      company.prevDemand = typeof company.demand === 'number' ? company.demand : (company.baseDemand || 100);
      const revenueSignal =
        (resourceScore - 1) * 0.9
        + (macroScore - 1) * 0.8
        + (eventShock - 1) * 0.7
        + (energyFactor - 1) * 1.2
        + (techFactor - 1) * 0.65
        + cycle * 0.85
        - Math.max(0, S.socialUnrest - 28) * 0.004;
      const demandTarget = clamp(
        (company.baseDemand || 100)
        + (resourceScore - 1) * 82
        + (macroScore - 1) * 76
        + (eventShock - 1) * 58
        + (energyFactor - 1) * 64
        + (techFactor - 1) * 52
        + (ownershipFactor - 1) * 24
        + (siteFactor - 1) * 34
        + cycle * 42
        + rand(-2.6, 2.6) * difficulty.priceNoise,
        35, 240
      );
      company.demand = clamp((company.prevDemand || company.baseDemand || 100) + (demandTarget - (company.prevDemand || company.baseDemand || 100)) * 0.24, 35, 240);
      const demandFactor = clamp(company.demand / Math.max(55, company.baseDemand || 100), 0.82, 1.26);
      maybeRotateRevenuePattern(company, revenueSignal);
      const revenuePattern = getCompanyRevenuePatternForce(company);
      const revenueWave = Math.sin(S.totalWeeks * (company.revenueWaveRate || 0.12) + (company.revenueWaveSeed || 0)) * company.baseRevenue * (company.revenueWaveAmp || 0.03);
      const revenueCenter = Math.max(
        240,
        company.baseRevenue
        * resourceScore
        * macroScore
        * eventShock
        * exportFactor
        * importFactor
        * energyFactor
        * techFactor
        * demandFactor
        * ownershipFactor
        * siteFactor
        * workerFactor
        * (1 + cycle * 0.45 + (company.revenueWaveDrift || 0) * 0.04)
      );
      const revenueSwingAmplitude = Math.max(
        80,
        company.baseRevenue * (
          0.022
          + Math.abs(revenuePattern.bias) * 0.012
          + (company.revenueWaveAmp || 0.03) * 0.85
          + company.volatility * 0.7
        )
      );
      let candidateRevenue = company.revenue
        + (revenueCenter - company.revenue) * 0.11
        + revenuePattern.shape * revenueSwingAmplitude
        + revenuePattern.bias * company.baseRevenue * 0.035
        + revenueWave
        + rand(-company.baseRevenue * 0.011 * difficulty.revenueNoise, company.baseRevenue * 0.011 * difficulty.revenueNoise);
      if(resourceScore < 0.82){
        candidateRevenue -= company.baseRevenue * (0.02 + (0.82 - resourceScore) * 0.11);
      } else if(resourceScore > 1.04){
        candidateRevenue += company.baseRevenue * Math.min(0.024, (resourceScore - 1.04) * 0.05);
      }
      if(startupRatio > 0 && resourceScore > 0.18){
        const startupRevenueFloor = company.baseRevenue * (0.92 + startupRatio * (0.07 + difficulty.startupGuard));
        if(candidateRevenue < startupRevenueFloor){
          candidateRevenue = startupRevenueFloor + rand(-company.baseRevenue * 0.008, company.baseRevenue * 0.008);
        }
      }
      company.revenue = Math.max(120, candidateRevenue);
      company.demandHist.push(company.demand);
      if(company.demandHist.length > 600) company.demandHist.shift();
      company.revenueHist.push(company.revenue);
      if(company.revenueHist.length > 600) company.revenueHist.shift();

      const revenueFactor = clamp(company.revenue / Math.max(1, company.baseRevenue), 0.55, 1.65);
      const demandPriceFactor = clamp(0.9 + (company.demand - (company.baseDemand || 100)) / 420, 0.88, 1.14);
      const passiveBubbleFactor = 1 + clamp((S.bubbleIndex || 0) / 320 * 0.22, 0, 0.22);
      const sectorBubbleFactor = S.bubbleActive && S.bubbleSector === company.sector ? 1.08 + S.bubbleIndex / 220 : 1;
      const bubbleFactor = passiveBubbleFactor * sectorBubbleFactor;
      const crashFactor = (!S.bubbleActive && (S.stability < 38 || S.marketShock > 0.5)) ? (startupRatio > 0 ? 0.88 : 0.72) * difficulty.crashBias : 1;
      const difficultyJitter = 1 + rand(-0.01, 0.01) * difficulty.priceNoise + (S.difficultyMode === 'hard' ? rand(-0.018, 0.024) : 0);
      const targetPrice = Math.max(100, Math.round((company.base || company.price || 100) * resourceScore * macroScore * eventShock * exportFactor * importFactor * techFactor * revenueFactor * demandPriceFactor * bubbleFactor * crashFactor * (1 + cycle + startupRatio * 0.022) * difficultyJitter));
      company.price = Math.max(100, Math.round(company.price + (targetPrice - company.price) * (0.08 * difficulty.priceResponse)));
      if(startupRatio > 0){
        const dynamicFloor = getCompanyDynamicFloor(company);
        const startupFloor = dynamicFloor + Math.max(0, (company.base || company.price || 100) - dynamicFloor) * (0.18 * startupRatio);
        if(company.price < startupFloor){
          company.price = Math.round(startupFloor + rand(-company.base * 0.01, company.base * 0.012));
        }
      }
      company.hist.push(company.price);
      if(company.hist.length > 600) company.hist.shift();
    });
    S.policyHeat = Math.max(0, S.policyHeat - 0.06);
    S.marketShock = Math.max(0, S.marketShock - 0.03);
  }

  function applyDividendFlows(){
    ensureCompanyModels();
    return 0;
  }

  function updateEnergySystem(){
    ensureCompanyModels();
    let production = S.nuclearPlants * 40;
    let demand = Math.max(80, S.pop * 0.028 + S.gdp * 0.018 + S.techLevel * 1.4 + companies.length * 0.9);
    S.publicWorks.forEach(work => {
      if(typeof work.damagedWeeks !== 'number') work.damagedWeeks = 0;
      if(work.damagedWeeks > 0){
        work.damagedWeeks--;
        if(work.damagedWeeks === 0) addEvent(`🔧 ${publicWorksCatalog[work.type]?.name || '公共事業'}が復旧`, 'good');
        return;
      }
      if(work.type === 'thermal') production += 34;
      if(work.type === 'hydro') production += 26;
      if(work.type === 'solar') production += 18;
      if(work.type === 'industrial'){ demand += 8; production += 4; }
      if(work.type === 'hospital'){ demand += 4; S.approval = Math.min(100, S.approval + 0.03); }
    });
    S.energy.production = Math.round(production * 10) / 10;
    S.energy.demand = Math.round(demand * 10) / 10;

    const gap = S.energy.production - S.energy.demand;
    if(gap < -8){
      S.gdp *= 0.9986;
      S.approval = Math.max(0, S.approval - 0.16);
      companies.filter(company => ['tech','auto','retail','space','telecom'].includes(company.sector)).forEach(company => {
        company.price = Math.max(100, Math.round(company.price * 0.994));
      });
      if(S.totalWeeks - S.lastPowerAlertWeek > 10){
        S.lastPowerAlertWeek = S.totalWeeks;
        addEvent('⚡ 電力不足で生産と生活が停滞', 'bad');
      }
    } else if(gap > 18){
      S.gdp *= 1.0007;
      S.approval = Math.min(100, S.approval + 0.04);
      companies.filter(company => ['energy','construction','space','tech'].includes(company.sector)).forEach(company => {
        company.price = Math.max(100, Math.round(company.price * 1.003));
      });
    }
  }

  function maybeTriggerEnhancedMajorEvent(){
    ensureCompanyModels();
    if(Math.random() >= 0.007) return;
    const pool = [
      {
        msg:'🌍 重大イベント: 世界金融危機が発生',
        apply(){
          S.marketShock += 0.8;
          S.gdp *= 0.965;
          S.bubbleIndex = Math.max(0, S.bubbleIndex - 10);
          companies.filter(company => ['finance','retail','auto','tech'].includes(company.sector)).forEach(company => company.price = Math.max(100, Math.round(company.price * 0.72)));
        }
      },
      {
        msg:'🦠 重大イベント: 新型感染症が急拡大',
        apply(){
          S.marketShock += 0.65;
          S.gdp *= 0.958;
          companies.filter(company => ['shipping','retail','food','auto'].includes(company.sector)).forEach(company => company.price = Math.max(100, Math.round(company.price * 0.68)));
          companies.filter(company => ['pharma','hospital'].includes(company.sector)).forEach(company => company.price = Math.max(100, Math.round(company.price * 1.15)));
        }
      },
      {
        msg:'🌊 重大イベント: 巨大地震と津波が発生',
        apply(){
          const regionKeys = Object.keys(regions).filter(key => !regions[key].isSea);
          const hitRegion = regionKeys[Math.floor(Math.random() * regionKeys.length)];
          const regionName = regions[hitRegion]?.name || '沿岸部';
          S.gdp *= 0.962;
          S.marketShock += 0.72;
          S.publicWorks.filter(work => work.region === hitRegion).forEach(work => { work.damagedWeeks = Math.max(work.damagedWeeks || 0, 14 + Math.floor(Math.random() * 18)); });
          S.companyFacilities.filter(site => site.region === hitRegion).forEach(site => { site.damagedWeeks = Math.max(site.damagedWeeks || 0, 10 + Math.floor(Math.random() * 14)); });
          companies.filter(company => company.region === hitRegion).forEach(company => company.price = Math.max(100, Math.round(company.price * 0.64)));
          companies.filter(company => company.sector === 'construction').forEach(company => company.price = Math.max(100, Math.round(company.price * 1.14)));
          addEvent(`📍 被災地域: ${regionName}`, 'bad');
        }
      },
      {
        msg:'🛢 重大イベント: エネルギー危機で輸入コストが急騰',
        apply(){
          S.marketShock += 0.55;
          if(resources.oil?.price) resources.oil.price *= 1.28;
          if(resources.gas?.price) resources.gas.price *= 1.22;
          companies.filter(company => ['retail','shipping','auto'].includes(company.sector)).forEach(company => company.price = Math.max(100, Math.round(company.price * 0.74)));
          companies.filter(company => company.sector === 'energy').forEach(company => company.price = Math.max(100, Math.round(company.price * 1.09)));
        }
      }
    ];
    const picked = pool[Math.floor(Math.random() * pool.length)];
    picked.apply();
    addEvent(picked.msg, 'bad');
  }

  function progressSpaceMissionFallback(beforeMissionKey){
    if(!S.space || !S.space.mission) return;
    const currentKey = `${S.space.mission.targetId || S.space.mission.kind}:${S.space.mission.remainingWeeks}`;
    if(beforeMissionKey && currentKey !== beforeMissionKey) return;
    if(typeof S.space.mission.remainingWeeks !== 'number') return;
    S.space.mission.remainingWeeks = Math.max(0, S.space.mission.remainingWeeks - 1);
    if(S.space.mission.remainingWeeks <= 0){
      const finishedMission = {...S.space.mission};
      S.space.mission = null;
      if(typeof resolveSpaceMission === 'function') resolveSpaceMission(finishedMission);
    }
  }

  getSaveMeta = function(slot){
    try{
      const raw = localStorage.getItem(getSaveStorageKey(slot));
      if(!raw) return null;
      const parsed = decodeSavePayload(raw);
      return parsed && parsed.meta ? {...parsed.meta, savedAt:parsed.savedAt} : null;
    }catch(error){
      return rawGetSaveMeta ? rawGetSaveMeta(slot) : null;
    }
  };

  performSave = function(slot, options={}){
    try{
      const payload = createSaveData(slot);
      S.lastUsedSaveSlot = slot;
      if(!options.preserveResumeSlot) setLastResumeSlot(slot);
      localStorage.setItem(getSaveStorageKey(slot), encodeSavePayload(payload));
      if(!options.silent) addEvent(S.language === 'en' ? `💾 Saved to ${getSaveLabel(slot, payload?.meta)}` : `💾 ${getSaveLabel(slot, payload?.meta)} に保存`, 'good');
      if(document.getElementById('actionModal').classList.contains('show') && document.getElementById('modalTitle').textContent.includes('セーブ')) openAction('save');
      return true;
    }catch(error){
      if(!options.silent) addEvent(S.language === 'en' ? '❌ Save failed: check storage or encryption.' : '❌ セーブ失敗: 保存領域または暗号化処理を確認してください', 'bad');
      return rawPerformSave ? rawPerformSave(slot, options) : false;
    }
  };

  window.loadGame = function(slot){
    try{
      const raw = localStorage.getItem(getSaveStorageKey(slot));
      if(!raw){ addEvent(S.language === 'en' ? '❌ No save data found.' : '❌ セーブデータがありません', 'bad'); return; }
      const parsed = decodeSavePayload(raw);
      applySaveData(parsed);
      S.lastUsedSaveSlot = slot;
      setLastResumeSlot(slot);
      ensureCompanyModels();
      updateHomeScreen();
      repairCoreUI();
      S.started = true;
      if(S.timeMode !== 'turn' && S.speed === 0) setSpeed(1);
      addEvent(S.language === 'en' ? `📂 Loaded ${getSaveLabel(slot, parsed?.meta)}` : `📂 ${getSaveLabel(slot, parsed?.meta)} を読込`, 'good');
      closeModal();
      if(document.getElementById('homeScreen')?.classList.contains('show')) closeHomeScreen();
    }catch(error){
      if(rawLoadGame) rawLoadGame(slot);
      else addEvent(S.language === 'en' ? '❌ Load failed: save data may be damaged.' : '❌ ロード失敗: データ破損の可能性があります', 'bad');
    }
  };

  if(rawBuyStock){
    window.buyStock = function(name, qty){
      ensureCompanyModels();
      const company = companies.find(item => item.name === name);
      if(!company) return;
      const owned = S.ownedStocks[name] || 0;
      const remaining = Math.max(0, company.sharesOutstanding - owned);
      if(remaining <= 0){
        addEvent(`🏢 ${name} はすでに100%保有済みです`, '');
        return;
      }
      rawBuyStock(name, Math.min(qty, remaining));
      const pct = ownershipRatio(company);
      if(pct >= 100 && !company.ownerEvent){
        company.ownerEvent = true;
        addEvent(`👑 ${name} を100%買収。研究・工場経営・施設配置が解禁`, 'good');
      }
    };
  }

  renderSpace = function(){
    ensureCompanyModels();
    if(!spaceBodies || !spaceBodies.length) return rawRenderSpace();
    const totalDiscoveries = Object.values(S.space.planets).filter(planet => planet.developed).length;
    let html = `<div class="section"><h3>🚀 宇宙開発局</h3>
      <div class="macro-grid">
        <div class="macro-card"><div class="macro-label">宇宙開発Lv</div><div class="macro-value">${S.space.level}</div><div style="font-size:8px;color:#aaa">Lv100で太陽到達計画を解禁</div></div>
        <div class="macro-card"><div class="macro-label">ロケットLv</div><div class="macro-value">${S.space.rocketLevel}</div><div style="font-size:8px;color:#aaa">成功率と移動週数に影響</div></div>
        <div class="macro-card"><div class="macro-label">探査済み惑星</div><div class="macro-value">${totalDiscoveries}</div><div style="font-size:8px;color:#aaa">惑星ごとに資源は完全分離管理</div></div>
        <div class="macro-card"><div class="macro-label">宇宙企業波及</div><div class="macro-value">${getSpaceCompanies().length}社</div><div style="font-size:8px;color:#aaa">株価・売上・技術力へ連動</div></div>
      </div>
      <div class="goal-card">
        <div class="goal-title"><span>🚀 ロケット開発</span><span>次: Lv${S.space.rocketLevel + 1}</span></div>
        <div class="goal-desc">高性能ロケットほど遠方探査が可能になります。費用は高く、国家技術力の土台も必要です。</div>
        <button class="btn btn-blue" onclick="developRocket()">開発 ${moneyCompact(getRocketDevelopmentCost(S.space.rocketLevel + 1))}</button>
      </div>`;

    if(S.space.mission){
      const missionBody = (spaceBodies || []).find(body => body.id === S.space.mission.targetId);
      const totalWeeks = S.space.mission.totalWeeks || S.space.mission.remainingWeeks || 1;
      const doneWeeks = totalWeeks - (S.space.mission.remainingWeeks || 0);
      const pct = clamp(doneWeeks / Math.max(1, totalWeeks), 0, 1);
      html += `<div class="goal-card">
        <div class="goal-title"><span>🛰 進行中ミッション</span><span>${missionBody?.name || S.space.mission.targetId}</span></div>
        <div class="goal-desc">${S.space.mission.kind === 'sun' ? '太陽到達計画' : '資源探査ミッション'} | 残り ${S.space.mission.remainingWeeks}週</div>
        <div class="goal-progress"><div class="fill" style="width:${Math.round(pct * 100)}%"></div></div>
      </div>`;
    }

    if(S.space.level >= 100 && !S.space.solarUnlocked){
      html += `<div class="goal-card">
        <div class="goal-title"><span>☀ 特別計画</span><span>解禁</span></div>
        <div class="goal-desc">宇宙開発Lv100到達。太陽到達計画で究極素材を狙えます。</div>
        <button class="btn btn-yellow" onclick="launchSpaceMission('sun','sun')">太陽到達計画を開始</button>
      </div>`;
    } else if(S.space.solarUnlocked){
      html += `<div class="headline-card good"><div class="meta">太陽到達</div><div class="body">太陽圏到達によって特殊エネルギー素材が解放されています。</div></div>`;
    }

    spaceBodies.forEach(body => {
      const state = getPlanetState(body.id);
      const materials = getPlanetMaterials(body);
      const rows = materials.map(material => {
        const total = state.total[material.id] || material.total || 0;
        const remaining = state.remaining[material.id] ?? total;
        return `<div class="trade-item"><span>${material.name}</span><span>${amountCompact(remaining)} / ${amountCompact(total)}</span></div>`;
      }).join('');
      const rocketReq = getRocketRequirement(body);
      const travelWeeks = getTravelWeeksForBody(body);
      const canLaunch = S.space.rocketLevel >= rocketReq && !S.space.mission;
      const cost = Math.round((body.cost || 9000 + rocketReq * 4200) * (body.id === 'moon' ? 1 : 1.08));
      html += `<div class="section">
        <h3>${body.icon || '🪐'} ${body.name}</h3>
        <div style="font-size:9px;color:#aaa;line-height:1.5;margin-bottom:5px">${body.desc || body.description || '個別の惑星資源を管理する探査先です。'}</div>
        <div class="detail-grid">
          <div class="dg-item"><div class="dg-label">必要ロケット</div><div class="dg-value">Lv${rocketReq}</div></div>
          <div class="dg-item"><div class="dg-label">移動時間</div><div class="dg-value">${travelWeeks}週</div></div>
        </div>
        <div style="margin-top:6px">${rows || '<div style="font-size:9px;color:#888">資源情報なし</div>'}</div>
        <div class="save-actions">
          <button class="btn btn-blue" onclick="launchSpaceMission('survey','${body.id}')" ${canLaunch ? '' : 'disabled'}>探査 ${moneyCompact(cost)}</button>
        </div>
      </div>`;
    });
    return html;
  };

  window.developRocket = function(){
    ensureCompanyModels();
    const nextLevel = S.space.rocketLevel + 1;
    const cost = getRocketDevelopmentCost(nextLevel);
    const techNeed = 10 + nextLevel * 6;
    if(S.money < cost){ addEvent('❌ ロケット開発資金が不足しています', 'bad'); return; }
    if(S.techLevel < techNeed){ addEvent(`❌ 国家技術力${techNeed}が必要です`, 'bad'); return; }
    S.money -= cost;
    S.space.rocketLevel = nextLevel;
    S.space.level = Math.max(S.space.level, nextLevel * 5);
    getSpaceCompanies().forEach(company => {
      company.techLevel = clamp(company.techLevel + 2.2, 0, 100);
      company.price = Math.max(100, Math.round(company.price * 1.035));
      company.revenue += cost * 0.04;
    });
    addEvent(`🚀 ロケットLv${nextLevel}を開発。遠方惑星への到達能力が向上`, 'good');
    updateUI();
  };

  window.launchSpaceMission = function(kind, bodyId){
    ensureCompanyModels();
    if(kind && !bodyId && spaceBodies.find(body => body.id === kind)){
      bodyId = kind;
      kind = 'survey';
    }
    if(kind !== 'survey' && kind !== 'sun'){
      if(rawLaunchSpaceMission) return rawLaunchSpaceMission(kind, bodyId);
    }
    if(S.space.mission){
      addEvent('❌ 進行中の宇宙ミッションがあります', 'bad');
      return;
    }
    if(kind === 'sun'){
      if(S.space.level < 100){ addEvent('❌ 宇宙開発Lv100で解禁されます', 'bad'); return; }
      const solarCost = 180000;
      if(S.money < solarCost){ addEvent('❌ 太陽到達計画の資金が足りません', 'bad'); return; }
      S.money -= solarCost;
      S.space.mission = {kind:'sun', targetId:'sun', totalWeeks:120, remainingWeeks:120};
      addEvent('☀ 太陽到達計画を開始。非常に長期の国家プロジェクトです', 'good');
      renderPanel();
      return;
    }
    const body = spaceBodies.find(item => item.id === bodyId);
    if(!body){
      if(rawLaunchSpaceMission) return rawLaunchSpaceMission(kind, bodyId);
      addEvent('❌ 宇宙目標が見つかりません', 'bad');
      return;
    }
    const requirement = getRocketRequirement(body);
    if(S.space.rocketLevel < requirement){ addEvent(`❌ ${body.name} へ行くにはロケットLv${requirement}が必要です`, 'bad'); return; }
    const travelWeeks = getTravelWeeksForBody(body);
    const cost = Math.round((body.cost || 9000 + requirement * 4200) * (1 + requirement * 0.08));
    if(S.money < cost){ addEvent('❌ 宇宙ミッション資金が不足しています', 'bad'); return; }
    S.money -= cost;
    S.space.rockets++;
    S.space.mission = {kind:'survey', targetId:body.id, totalWeeks:travelWeeks, remainingWeeks:travelWeeks, cost};
    addEvent(`🛰 ${body.name} への長期探査を開始（${travelWeeks}週）`, 'good');
    renderPanel();
  };

  resolveSpaceMission = function(mission){
    ensureCompanyModels();
    if(!mission || !mission.targetId){
      if(rawResolveSpaceMission) return rawResolveSpaceMission(mission);
      return;
    }
    if(mission.kind === 'sun'){
      S.space.solarUnlocked = true;
      S.space.successes++;
      S.space.level = Math.max(100, S.space.level);
      ensurePlanetGlobalResource({id:'solar_plasma', name:'ソーラープラズマ', total:4000, price:4200, color:'#ffd54f', unit:'kg'}, {id:'sun'});
      resources.solar_plasma.amount += 220;
      resources.solar_plasma.max = Math.max(resources.solar_plasma.max, 5000);
      getSpaceCompanies().forEach(company => {
        company.techLevel = clamp(company.techLevel + 6, 0, 100);
        company.revenue += 1400;
        company.price = Math.max(100, Math.round(company.price * 1.08));
      });
      addEvent('☀ 太陽到達計画が成功。究極エネルギー素材「ソーラープラズマ」を回収', 'good');
      return;
    }
    const body = spaceBodies.find(item => item.id === mission.targetId);
    if(!body){
      if(rawResolveSpaceMission) return rawResolveSpaceMission(mission);
      return;
    }
    const planet = getPlanetState(body.id);
    const successChance = clamp(0.35 + S.space.rocketLevel * 0.08 + S.techLevel * 0.004 + S.space.level * 0.002 - getRocketRequirement(body) * 0.05, 0.28, 0.92);
    if(Math.random() > successChance){
      S.space.failures++;
      S.approval = Math.max(0, S.approval - 1.8);
      getSpaceCompanies().forEach(company => {
        company.price = Math.max(100, Math.round(company.price * 0.95));
      });
      addEvent(`💥 ${body.name} 探査に失敗。遠方宇宙の難しさが露呈した`, 'bad');
      return;
    }
    const materials = getPlanetMaterials(body);
    let totalExtracted = 0;
    const extractedNames = [];
    materials.forEach(material => {
      const total = planet.total[material.id] || material.total || 0;
      const remaining = planet.remaining[material.id] ?? total;
      const extract = Math.min(remaining, Math.max(5, Math.round(total * (body.id === 'moon' ? 0.04 : 0.025))));
      if(extract <= 0) return;
      planet.remaining[material.id] = remaining - extract;
      totalExtracted += extract;
      extractedNames.push(`${material.name} +${amountCompact(extract)}`);
      const resource = ensurePlanetGlobalResource(material, body);
      resource.amount += extract;
      resource.max = Math.max(resource.max, resource.amount * 2, total);
      if(!resource.priceHist) resource.priceHist = [resource.price || material.price || 120];
    });
    planet.developed = true;
    S.space.successes++;
    S.space.level = Math.min(100, S.space.level + 4 + getRocketRequirement(body));
    S.space.resources += totalExtracted;
    getSpaceCompanies().forEach(company => {
      company.revenue += totalExtracted * 0.8 + 240;
      company.techLevel = clamp(company.techLevel + 1.8, 0, 100);
      company.price = Math.max(100, Math.round(company.price * (1.03 + getRocketRequirement(body) * 0.004)));
    });
    if(totalExtracted > 0) addEvent(`🪐 ${body.name} から資源回収成功: ${extractedNames.join(', ')}`, 'good');
    else addEvent(`🪐 ${body.name} は枯渇しており、新規資源は得られなかった`, 'bad');
    if(S.space.level >= 100 && !S.space.solarUnlocked) addEvent('☀ 宇宙開発Lv100到達。太陽到達計画が解禁された', 'good');
  };

  if(rawOpenCompanyDetail){
    window.openCompanyDetail = function(name){
      rawOpenCompanyDetail(name);
      const company = companies.find(item => item.name === name);
      if(!company || ownershipRatio(company) < 100) return;
      const host = document.getElementById('modalContent');
      if(!host || host.querySelector('[data-owner-tools]')) return;
      const facilities = S.companyFacilities.filter(item => item.companyName === name);
      const facilityHtml = facilities.length
        ? facilities.map(item => `<div class="trade-item"><span>${item.label} / ${regions[item.region]?.name || item.region} / ${item.workers}人</span><span>${item.damagedWeeks > 0 ? `復旧${item.damagedWeeks}週` : '稼働中'}</span></div>`).join('')
        : '<div class="compact-note">まだ自社施設はありません。</div>';
      host.insertAdjacentHTML('beforeend', `<div class="section" data-owner-tools>
        <h3>🏭 オーナー経営</h3>
        <div class="compact-note">100%保有中のため、施設建設と作業員調整が可能です。</div>
        <div class="save-actions" style="margin-top:8px">
          <button class="btn btn-blue" onclick="beginCompanyFacilityPlacement('${name}','factory')">工場を配置</button>
          <button class="btn btn-blue" onclick="beginCompanyFacilityPlacement('${name}','hospital')">病院を配置</button>
          <button class="btn btn-blue" onclick="beginCompanyFacilityPlacement('${name}','lab')">研究所を配置</button>
          <button class="btn" onclick="adjustCompanyWorkers('${name}', 40)">作業員 +40</button>
          <button class="btn" onclick="adjustCompanyWorkers('${name}', -40)">作業員 -40</button>
        </div>
        <div style="margin-top:8px">${facilityHtml}</div>
      </div>`);
    };
  }

  window.beginCompanyFacilityPlacement = beginCompanyFacilityPlacement;
  window.adjustCompanyWorkers = adjustCompanyWorkers;

  drawMap = function(){
    rawDrawMap();
    ensureCompanyModels();
    if(S.currentMapMode === 'space') return;
    S.companyFacilities.forEach(facility => {
      const point = worldToCanvas(facility.x, facility.y);
      ctx.beginPath();
      ctx.arc(point.x, point.y, Math.max(8, 10 * S.mapZoom), 0, Math.PI * 2);
      ctx.fillStyle = facility.type === 'factory' ? 'rgba(144,202,249,.92)' : facility.type === 'hospital' ? 'rgba(239,154,154,.92)' : 'rgba(206,147,216,.92)';
      ctx.fill();
      ctx.strokeStyle = 'rgba(255,255,255,.6)';
      ctx.lineWidth = 1.5;
      ctx.stroke();
      ctx.fillStyle = '#fff';
      ctx.font = `${10 * S.mapZoom}px sans-serif`;
      ctx.textAlign = 'center';
      ctx.fillText(facility.type === 'factory' ? 'F' : facility.type === 'hospital' ? 'H' : 'L', point.x, point.y + 3);
    });
    if(S.draggingFacility){
      const point = worldToCanvas(S.draggingFacility.x, S.draggingFacility.y);
      ctx.globalAlpha = 0.85;
      ctx.beginPath();
      ctx.arc(point.x, point.y, Math.max(9, 12 * S.mapZoom), 0, Math.PI * 2);
      ctx.fillStyle = S.draggingFacility.color || '#fff176';
      ctx.fill();
      ctx.globalAlpha = 1;
      ctx.fillStyle = '#fff';
      ctx.font = `${11 * S.mapZoom}px sans-serif`;
      ctx.textAlign = 'left';
      ctx.fillText(`${S.draggingFacility.companyName} ${S.draggingFacility.label}`, point.x + 12, point.y - 8);
    }
  };

  gameArea.addEventListener('mousemove', e => {
    if(!S.draggingFacility || S.currentMapMode === 'space') return;
    const rect = canvas.getBoundingClientRect();
    const canvasX = (e.clientX - rect.left) / rect.width * mapW;
    const canvasY = (e.clientY - rect.top) / rect.height * mapH;
    const worldPos = canvasToWorld(canvasX, canvasY);
    S.draggingFacility.x = worldPos.x;
    S.draggingFacility.y = worldPos.y;
    drawMap();
  }, true);

  gameArea.addEventListener('click', e => {
    if(!S.draggingFacility || S.currentMapMode === 'space') return;
    e.stopImmediatePropagation();
    const rect = canvas.getBoundingClientRect();
    const canvasX = (e.clientX - rect.left) / rect.width * mapW;
    const canvasY = (e.clientY - rect.top) / rect.height * mapH;
    const worldPos = canvasToWorld(canvasX, canvasY);
    placeCompanyFacility(S.draggingFacility, worldPos);
    S.draggingFacility = null;
    drawMap();
    updateUI();
  }, true);

  function advanceWorldAI(){
    ensureForeignCompanies();
    tradePartners.forEach((partner, index) => {
      partner.economy = clamp(partner.economy + rand(-2.2, 2.4) + (partner.relation < 40 ? 0.6 : 0), 15, 100);
      partner.cyberPower = clamp(partner.cyberPower + rand(-1.0, 1.6), 10, 100);
      if(Math.random() < 0.06){
        partner.relation = clamp(partner.relation + rand(-4, 3), 0, 100);
      }
      if(Math.random() < 0.025 && partner.cyberPower > S.cyberPower + 8){
        const damage = 220 + partner.cyberPower * 6;
        S.money = Math.max(0, S.money - damage);
        S.worldLog.unshift(`${partner.name} がサイバー攻撃を実施し、${moneyCompact(damage)}の損失`);
        addEvent(`💻 ${partner.name} からサイバー攻撃`, 'bad');
      }
      const partnerCompanies = S.foreignCompanies.filter(company => company.partnerIndex === index);
      partnerCompanies.forEach(company => {
        company.prevPrice = company.price;
        const fxFactor = S.yenValue < 100 ? 1.02 : 0.99;
        const targetRevenue = Math.max(200, company.baseRevenue * (partner.economy / 70) * (company.techLevel / 35) * fxFactor * rand(0.92, 1.08));
        company.revenue += (targetRevenue - company.revenue) * 0.16;
        company.price = Math.max(100, Math.round(company.price + ((company.base * (partner.economy / 70) * rand(0.88, 1.12)) - company.price) * 0.1));
        company.revenueHist.push(company.revenue);
        company.hist.push(company.price);
        if(company.revenueHist.length > 520) company.revenueHist.shift();
        if(company.hist.length > 520) company.hist.shift();
      });
    });
    S.foreignDividend = 0;
    S.worldLog = S.worldLog.slice(0, 12);
  }

  renderPanel = function(){
    const content = document.getElementById('panelContent');
    if(S.currentPanel === 'world'){
      if(content) content.innerHTML = translatePanelMarkup(renderWorld());
      return;
    }
    rawRenderPanel();
  };

  gameTick = function(){
    ensureCompanyModels();
    const missionBefore = S.space && S.space.mission ? `${S.space.mission.targetId || S.space.mission.kind}:${S.space.mission.remainingWeeks}` : '';
    rawGameTick();
    ensureCompanyModels();
    const moneyAfterBaseTick = S.money;

    maybeTriggerEnhancedMajorEvent();
    updateEnergySystem();

    S.techLevel = clamp(
      S.techLevel
        + S.annualBudget.technology / 180000
        + S.space.level * 0.004
        + S.discoveredMaterials.length * 0.028
        - Math.max(0, S.inflation - 5) * 0.01,
      0,
      100
    );

    S.approval = clamp(
      S.approval
        + S.annualBudget.education / 420000
        + S.publicWorks.filter(work => work.type === 'hospital' && !(work.damagedWeeks > 0)).length * 0.03,
      0,
      100
    );

    smoothCompanyEconomics();
    const dividendIncome = applyDividendFlows();
    progressSpaceMissionFallback(missionBefore);
    if(S.totalWeeks % 2 === 0) advanceWorldAI();
    const breakdown = {
      tax:0,budget:0,trade:0,dividend:0,space:0,interest:0,principal:0,companySupport:0,publicRevenue:0,operations:0,socialSecurity:0,healthcare:0,childcare:0,administration:0,adjustments:0,
      ...(S.weeklyBreakdown || {}),
    };
    const foreignDividend = S.totalWeeks % 2 === 0 ? (S.foreignDividend || 0) : 0;
    breakdown.dividend = (breakdown.dividend || 0) + dividendIncome + foreignDividend;
    const moneyDeltaAfterBase = S.money - moneyAfterBaseTick;
    const categorizedDelta = dividendIncome + foreignDividend;
    breakdown.adjustments = (breakdown.adjustments || 0) + (moneyDeltaAfterBase - categorizedDelta);
    S.weeklyBreakdown = breakdown;

    if(S.approval <= 0 && !S.sandboxMode){
      setSpeed(0);
      openHomeScreen('支持率が0%となり内閣は崩壊しました。セーブからの再開か新規開始を選んでください。');
    }
    rawUpdateUI();
  };

  updateUI = function(){
    ensureCompanyModels();
    rawUpdateUI();
    updateHomeScreen();
    updateSpeedButtons();
  };

  window.openTopStatDetail = openTopStatDetail;

  const topBarElement = document.getElementById('topBar');
  if(topBarElement && !topBarElement.dataset.detailBound){
    topBarElement.dataset.detailBound = '1';
    topBarElement.addEventListener('click', event => {
      const stat = event.target.closest('.stat.clickable');
      if(!stat || !stat.dataset.stat) return;
      handleTopStatCardClick(stat.dataset.stat);
    });
  }

  gameArea.addEventListener('contextmenu', e => {
    if(S.mapMode === 'space') e.preventDefault();
  });

  closeHomeScreen = function(){
    if(rawCloseHomeScreen) rawCloseHomeScreen();
    else{
      const screen = document.getElementById('homeScreen');
      if(screen) screen.classList.remove('show');
      S.started = true;
      if(S.timeMode !== 'turn' && S.speed === 0) setSpeed(1);
    }
    repairCoreUI();
    if(S.timeMode !== 'turn' && S.speed === 0) setSpeed(1);
    lastTick = performance.now();
  };

  window.startFromHome = function(){
    closeHomeScreen();
  };

  window.continueFromHome = function(){
    const slot = getMostRecentSaveSlot();
    if(!slot || !getSaveMeta(slot)){
      updateHomeScreen(S.language === 'en' ? 'There is no recent save yet.' : '前回の続きはまだありません。');
      return;
    }
    loadGame(slot);
  };

  const panelTabs = document.getElementById('panelTabs');
  if(panelTabs && !panelTabs.querySelector('[data-panel="world"]')){
    const worldBtn = document.createElement('button');
    worldBtn.className = 'btn';
    worldBtn.dataset.panel = 'world';
    worldBtn.textContent = '世界';
    panelTabs.insertBefore(worldBtn, panelTabs.querySelector('[data-panel="compare"]') || panelTabs.firstChild);
  }

  repairCoreUI();
  updateHomeScreen();
  updateUI();
})();

(function(){
  const TUTORIAL_STORAGE_KEY = `${STORAGE_PREFIX}:tutorial-complete`;
  const TUTORIAL_FLOW = [
    {id:'welcome', manual:true},
    {id:'prefecture', event:'prefecture_open'},
    {id:'company', event:'company_open'},
    {id:'stock', event:'stock_bought'},
    {id:'trade', event:'trade_started'},
    {id:'budget', event:'budget_applied'},
    {id:'construction', event:'construction_started'},
    {id:'time', event:'time_advanced'},
    {id:'wrapup', manual:true},
  ];

  const TUTORIAL_COPY = {
    ja:{
      homeButton:'📘 チュートリアル',
      homeReplay:'📘 チュートリアルをもう一度',
      loader:'チュートリアルを準備中...',
      cardTitle:'はじめてガイド',
      badge:(current,total) => `STEP ${current}/${total}`,
      reopen:'説明を見る',
      jump:'関連画面へ',
      skip:'この手順をスキップ',
      continue:'次へ',
      close:'閉じる',
      exit:'終了',
      cardCurrent:'今やること',
      cardHint:'ヒント',
      completedTitle:'🎓 チュートリアル完了',
      completedBody:'基本操作をひと通り体験できました。このまま本編を続けて、日本経済の運営を自由に進められます。',
      completedNote:'右パネル、年間予算、貿易、公共事業、ターン進行を使えば、以後は自分のペースで遊べます。',
      completedButton:'本編を続ける',
      exited:'📘 チュートリアルを終了しました。',
      stepCleared:'📘 チュートリアル達成',
      stepSkipped:'⏭ チュートリアルの手順をスキップ',
      steps:{
        welcome:{
          title:'ようこそ',
          body:'この専用モードでは、地図、企業、貿易、予算、建設、時間進行の基本を順番に覚えます。ゲーム速度は最初からターン制にしてあるので、落ち着いて試せます。',
          hint:'右下の「⏭進む」で1週ずつ進められます。迷ったらカードの「説明を見る」を押してください。'
        },
        prefecture:{
          title:'都道府県を開く',
          body:'日本マップで任意の都道府県を左クリックし、県の詳細を開いてください。県ごとの企業や資源を見るのが最初の入口です。',
          hint:'北海道でも東京でもOKです。右クリックは地方企業一覧なので、今回は左クリックを使ってみましょう。'
        },
        company:{
          title:'企業を1社調べる',
          body:'県の詳細や市場タブから企業を1社開いてみてください。株価、必要素材、好感度、技術方針など、その会社の基本情報が見えます。',
          hint:'市場タブなら会社一覧から、県詳細ならその県の企業一覧から開けます。'
        },
        stock:{
          title:'最初の株を買う',
          body:'気になる企業を1株以上買ってみましょう。保有すると配当や研究、企業経営への関わりが見えてきます。',
          hint:'最初は1株で十分です。資金と残株が足りない時は別の会社でも構いません。'
        },
        trade:{
          title:'輸入ルートを1本作る',
          body:'貿易タブを開いて、資源の輸入便を1本契約してください。資源が不足すると企業の売上や株価に影響するので、ここは重要です。',
          hint:'不足している資源は貿易タブ上部にまとまっているので、そこから始めるとわかりやすいです。'
        },
        budget:{
          title:'年間予算を確定する',
          body:'年間予算を1回決めてみましょう。軍事、教育、技術、企業支援の配分で国の伸び方が変わります。',
          hint:'今年の予算がまだ未設定なら、どれかを少し動かして確定するだけで進みます。'
        },
        construction:{
          title:'開発か公共事業を1つ始める',
          body:'地域開発を1回押すか、公共事業を1つ着工してみましょう。地図上の変化と企業への波及を体験する工程です。',
          hint:'手軽なのは開発タブの地域開発です。公共事業は配置が必要なので、慣れてきたらそちらでも大丈夫です。'
        },
        time:{
          title:'1ターン進める',
          body:'右下の「⏭進む」で1週進めてください。ターンを進めると予算、貿易、企業売上、イベント判定がまとめて反映されます。',
          hint:'ターン制では、考える→進める→結果を見る、を繰り返します。低スペック環境でも遊びやすいモードです。'
        },
        wrapup:{
          title:'締めくくり',
          body:'ここまでで、地図、企業、貿易、予算、開発、時間進行の基本を一通り触れました。下のボタンで本編に戻って、そのまま続けられます。',
          hint:'次は研究、法律、戦力、宇宙、災害などを少しずつ触っていくのがおすすめです。'
        },
      }
    },
    en:{
      homeButton:'📘 Tutorial',
      homeReplay:'📘 Replay Tutorial',
      loader:'Preparing the tutorial...',
      cardTitle:'Beginner Guide',
      badge:(current,total) => `STEP ${current}/${total}`,
      reopen:'Open Briefing',
      jump:'Open Related View',
      skip:'Skip This Step',
      continue:'Continue',
      close:'Close',
      exit:'Exit',
      cardCurrent:'Current Goal',
      cardHint:'Hint',
      completedTitle:'🎓 Tutorial Complete',
      completedBody:'You have now gone through the core flow. You can keep playing right away and manage Japan at your own pace.',
      completedNote:'Use the right panel, annual budget, trade, public works, and turn advance to continue the full game.',
      completedButton:'Continue Playing',
      exited:'📘 Tutorial ended.',
      stepCleared:'📘 Tutorial cleared',
      stepSkipped:'⏭ Tutorial step skipped',
      steps:{
        welcome:{
          title:'Welcome',
          body:'This dedicated mode teaches the core loop in order: map, companies, trade, budget, construction, and time progression. The game starts in turn mode so you can move calmly.',
          hint:'Use “⏭ Advance” in the bottom bar to process one week at a time. If you get lost, open the briefing again from the guide card.'
        },
        prefecture:{
          title:'Open a Prefecture',
          body:'Left-click any prefecture on the Japan map to open its detail view. Prefecture details are your first doorway into companies and resources.',
          hint:'Any prefecture works. Right-click opens the regional company list, so use a normal left-click for this step.'
        },
        company:{
          title:'Inspect One Company',
          body:'Open any company from a prefecture view or from the market panel. There you can check price, required materials, favorability, and technology direction.',
          hint:'You can open a company either from the prefecture modal or from the market tab.'
        },
        stock:{
          title:'Buy Your First Share',
          body:'Buy at least one share in any company. Owning stock is how you start interacting with dividends, research, and direct management.',
          hint:'One share is enough. If one company is not convenient, pick any other company.'
        },
        trade:{
          title:'Create One Import Route',
          body:'Open the trade panel and sign one import route. Resources feed directly into company output, so trade is one of the most important systems.',
          hint:'The trade panel already highlights shortage resources near the top, so starting there is usually easiest.'
        },
        budget:{
          title:'Approve an Annual Budget',
          body:'Set your annual budget once. Military, education, technology, and company support change the growth path of the country.',
          hint:'If this year is still unset, even a small adjustment followed by approval will complete the step.'
        },
        construction:{
          title:'Start Development or Public Works',
          body:'Either trigger one regional development action or begin one public work. This is how you shape the map and push sector growth.',
          hint:'Regional development is the quickest option. Public works are also valid if you want to place them on the map.'
        },
        time:{
          title:'Advance One Turn',
          body:'Use “⏭ Advance” in the bottom bar to process one week. That applies budgets, trade flows, company revenue, and event checks.',
          hint:'Turn mode is especially useful on low-spec PCs or phones because the heavy processing happens only when you press advance.'
        },
        wrapup:{
          title:'Wrap Up',
          body:'You have now touched the core loop: map, companies, trade, budget, development, and time progression. Use the button below to continue into the full game.',
          hint:'Next good systems to explore are research, laws, military, space, and disaster control.'
        },
      }
    }
  };

  function ensureTutorialState(){
    if(typeof S.tutorialActive !== 'boolean') S.tutorialActive = false;
    if(typeof S.tutorialStep !== 'number') S.tutorialStep = 0;
    if(typeof S.tutorialStartedWeek !== 'number') S.tutorialStartedWeek = S.totalWeeks || 0;
    if(typeof S.tutorialCompleted !== 'boolean'){
      try{
        S.tutorialCompleted = localStorage.getItem(TUTORIAL_STORAGE_KEY) === '1';
      } catch(error){
        S.tutorialCompleted = false;
      }
    }
  }

  function getTutorialLocale(){
    return TUTORIAL_COPY[S.language] || TUTORIAL_COPY.ja;
  }

  function getTutorialSteps(){
    const copy = getTutorialLocale();
    return TUTORIAL_FLOW.map(step => ({...step, ...(copy.steps[step.id] || {})}));
  }

  function getCurrentTutorialStep(){
    ensureTutorialState();
    const steps = getTutorialSteps();
    return steps[Math.max(0, Math.min(S.tutorialStep, steps.length - 1))] || null;
  }

  function ensureTutorialStyle(){
    if(document.getElementById('tutorialRuntimeStyle')) return;
    const style = document.createElement('style');
    style.id = 'tutorialRuntimeStyle';
    style.textContent = `
      .tutorial-card{
        position:fixed;left:12px;bottom:76px;width:min(360px,calc(100vw - 24px));z-index:190;
        display:none;padding:14px 14px 12px;border-radius:18px;
        background:linear-gradient(180deg,rgba(14,22,42,.94),rgba(11,17,31,.94));
        border:1px solid rgba(255,255,255,.08);box-shadow:0 18px 40px rgba(0,0,0,.34);
        backdrop-filter:blur(14px);
      }
      .tutorial-card.show{display:block;}
      .tutorial-card .tutorial-top{display:flex;justify-content:space-between;align-items:flex-start;gap:10px;margin-bottom:10px;}
      .tutorial-card .tutorial-top .title{font-size:15px;font-weight:800;color:#eff6ff;}
      .tutorial-card .tutorial-top .badge{padding:4px 8px;border-radius:999px;background:rgba(79,195,247,.12);border:1px solid rgba(79,195,247,.18);font-size:11px;color:#c8ecff;white-space:nowrap;}
      .tutorial-card .tutorial-step-title{font-size:18px;font-weight:800;color:#fff;margin-bottom:8px;line-height:1.3;}
      .tutorial-card .tutorial-current-label,.tutorial-card .tutorial-hint-label{font-size:11px;text-transform:uppercase;letter-spacing:.08em;color:#8eb7df;margin-bottom:4px;}
      .tutorial-card .tutorial-body{font-size:12px;line-height:1.7;color:#d7e5f7;}
      .tutorial-card .tutorial-hint{margin-top:10px;padding:10px 11px;border-radius:12px;background:rgba(255,255,255,.03);border:1px solid rgba(255,255,255,.05);}
      .tutorial-card .tutorial-progress{height:8px;border-radius:999px;background:rgba(255,255,255,.06);overflow:hidden;margin:12px 0 10px;}
      .tutorial-card .tutorial-progress > span{display:block;height:100%;background:linear-gradient(90deg,#4fc3f7,#53de88);border-radius:999px;}
      .tutorial-card .tutorial-actions{display:flex;flex-wrap:wrap;gap:7px;}
      .tutorial-card .tutorial-actions .btn{font-size:12px;padding:6px 10px;}
      .tutorial-modal-step{display:flex;align-items:center;justify-content:space-between;gap:10px;margin-bottom:10px;flex-wrap:wrap;}
      .tutorial-modal-step .badge{padding:4px 8px;border-radius:999px;background:rgba(79,195,247,.12);font-size:11px;color:#d4efff;border:1px solid rgba(79,195,247,.16);}
      .tutorial-modal-step-title{font-size:21px;font-weight:800;color:#fff;margin-bottom:10px;}
      .tutorial-modal-grid{display:grid;grid-template-columns:1fr;gap:10px;}
      .tutorial-modal-box{padding:12px;border-radius:12px;background:rgba(255,255,255,.03);border:1px solid rgba(255,255,255,.06);}
      .tutorial-modal-box .label{font-size:11px;text-transform:uppercase;letter-spacing:.08em;color:#9eb7dc;margin-bottom:6px;}
      .tutorial-modal-box .body{font-size:13px;line-height:1.8;color:#dce8f8;}
      .home-actions .tutorial-entry-btn{background:linear-gradient(135deg,rgba(79,195,247,.16),rgba(83,222,136,.15));border-color:rgba(92,210,255,.42);color:#eff9ff;}
      .home-actions .tutorial-entry-btn:hover{background:linear-gradient(135deg,#4fc3f7,#53de88);color:#03111f;}
      body.mode-world-map .tutorial-card{
        background:linear-gradient(180deg,rgba(6,16,10,.96),rgba(4,12,8,.94));
        border-color:rgba(111,255,171,.14);
      }
      body.mode-world-map .tutorial-card .tutorial-top .badge{background:rgba(111,255,171,.12);border-color:rgba(111,255,171,.16);color:#e7fff0;}
      @media (max-width: 860px){
        .tutorial-card{left:10px;right:10px;width:auto;bottom:74px;}
      }
    `;
    document.head.appendChild(style);
  }

  function ensureTutorialCard(){
    ensureTutorialStyle();
    let card = document.getElementById('tutorialCard');
    if(card) return card;
    card = document.createElement('div');
    card.id = 'tutorialCard';
    card.className = 'tutorial-card';
    document.body.appendChild(card);
    return card;
  }

  function ensureTutorialHomeButton(){
    const heroBtn = document.getElementById('homeHeroTutorialBtn');
    if(heroBtn){
      heroBtn.textContent = S.tutorialCompleted ? getTutorialLocale().homeReplay : getTutorialLocale().homeButton;
      return heroBtn;
    }
    return null;
  }

  function getTutorialProgress(stepIndex){
    const steps = getTutorialSteps();
    const total = Math.max(1, steps.length);
    return clamp(((stepIndex + 1) / total) * 100, 0, 100);
  }

  function renderTutorialCard(){
    ensureTutorialState();
    ensureTutorialHomeButton();
    const card = ensureTutorialCard();
    const homeScreen = document.getElementById('homeScreen');
    if(!S.tutorialActive || !S.started || homeScreen?.classList.contains('show')){
      card.classList.remove('show');
      card.innerHTML = '';
      return;
    }
    const copy = getTutorialLocale();
    const steps = getTutorialSteps();
    const step = getCurrentTutorialStep();
    if(!step){
      card.classList.remove('show');
      card.innerHTML = '';
      return;
    }
    const progress = getTutorialProgress(S.tutorialStep);
    const badge = copy.badge(Math.min(steps.length, S.tutorialStep + 1), steps.length);
    const primaryLabel = step.manual ? (step.id === 'wrapup' ? copy.completedButton : copy.continue) : copy.skip;
    const primaryAction = step.manual ? 'tutorialContinueStep()' : 'tutorialSkipStep()';
    card.innerHTML = `
      <div class="tutorial-top">
        <div class="title">📘 ${copy.cardTitle}</div>
        <div class="badge">${badge}</div>
      </div>
      <div class="tutorial-step-title">${step.title}</div>
      <div class="tutorial-current-label">${copy.cardCurrent}</div>
      <div class="tutorial-body">${step.body}</div>
      <div class="tutorial-hint">
        <div class="tutorial-hint-label">${copy.cardHint}</div>
        <div class="tutorial-body">${step.hint}</div>
      </div>
      <div class="tutorial-progress"><span style="width:${progress}%"></span></div>
      <div class="tutorial-actions">
        <button class="btn btn-blue btn-sm" onclick="tutorialOpenStep()">${copy.reopen}</button>
        ${step.id !== 'welcome' && step.id !== 'wrapup' ? `<button class="btn btn-sm" onclick="tutorialJumpToContext()">${copy.jump}</button>` : ''}
        <button class="btn btn-yellow btn-sm" onclick="${primaryAction}">${primaryLabel}</button>
        <button class="btn btn-sm" onclick="tutorialExit()">${copy.exit}</button>
      </div>
    `;
    card.classList.add('show');
  }

  function openTutorialStepModal(){
    ensureTutorialState();
    if(!S.tutorialActive) return;
    const step = getCurrentTutorialStep();
    if(!step) return;
    const copy = getTutorialLocale();
    const steps = getTutorialSteps();
    const modal = document.getElementById('actionModal');
    const titleEl = document.getElementById('modalTitle');
    const contentEl = document.getElementById('modalContent');
    if(!modal || !titleEl || !contentEl) return;
    titleEl.textContent = `📘 ${copy.cardTitle}`;
    modal.dataset.action = 'tutorial';
    const primaryLabel = step.manual ? (step.id === 'wrapup' ? copy.completedButton : copy.continue) : copy.skip;
    const primaryAction = step.manual ? 'tutorialContinueStep()' : 'tutorialSkipStep()';
    contentEl.innerHTML = `
      <div class="tutorial-modal-step">
        <div class="badge">${copy.badge(Math.min(steps.length, S.tutorialStep + 1), steps.length)}</div>
        <div class="pill">${formatWeekText()}</div>
      </div>
      <div class="tutorial-modal-step-title">${step.title}</div>
      <div class="tutorial-modal-grid">
        <div class="tutorial-modal-box">
          <div class="label">${copy.cardCurrent}</div>
          <div class="body">${step.body}</div>
        </div>
        <div class="tutorial-modal-box">
          <div class="label">${copy.cardHint}</div>
          <div class="body">${step.hint}</div>
        </div>
      </div>
      <div class="save-actions" style="margin-top:12px">
        ${step.id !== 'welcome' && step.id !== 'wrapup' ? `<button class="btn btn-blue" onclick="tutorialJumpToContext()">${copy.jump}</button>` : ''}
        <button class="btn btn-yellow" onclick="${primaryAction}">${primaryLabel}</button>
        <button class="btn" onclick="closeModal()">${copy.close}</button>
      </div>
    `;
    modal.classList.add('show');
  }

  function markTutorialCompleted(){
    ensureTutorialState();
    S.tutorialActive = false;
    S.tutorialCompleted = true;
    try{ localStorage.setItem(TUTORIAL_STORAGE_KEY, '1'); }catch(error){}
    renderTutorialCard();
    ensureTutorialHomeButton();
  }

  function completeTutorialFlow(){
    markTutorialCompleted();
    addEvent(getTutorialLocale().completedTitle, 'good');
    const copy = getTutorialLocale();
    document.getElementById('modalTitle').textContent = copy.completedTitle;
    document.getElementById('modalContent').innerHTML = `
      <div class="section">
        <div class="compact-note" style="font-size:13px;line-height:1.8">${copy.completedBody}</div>
        <div class="compact-note" style="margin-top:10px">${copy.completedNote}</div>
        <div class="save-actions" style="margin-top:14px">
          <button class="btn btn-green" onclick="closeModal()">${copy.completedButton}</button>
        </div>
      </div>
    `;
    document.getElementById('actionModal').dataset.action = 'tutorial';
    document.getElementById('actionModal').classList.add('show');
  }

  function setTutorialStep(nextIndex, options={}){
    ensureTutorialState();
    const steps = getTutorialSteps();
    S.tutorialStep = clamp(nextIndex, 0, steps.length - 1);
    S.tutorialStartedWeek = S.totalWeeks || 0;
    renderTutorialCard();
    if(options.openModal) openTutorialStepModal();
  }

  function advanceTutorialStep(options={}){
    ensureTutorialState();
    const steps = getTutorialSteps();
    if(S.tutorialStep >= steps.length - 1){
      completeTutorialFlow();
      return;
    }
    setTutorialStep(S.tutorialStep + 1, options);
  }

  function handleTutorialAction(action){
    ensureTutorialState();
    if(!S.tutorialActive) return;
    const step = getCurrentTutorialStep();
    if(!step || step.manual || step.event !== action) return;
    addEvent(`${getTutorialLocale().stepCleared}: ${step.title}`, 'good');
    const shouldOpenFinish = S.tutorialStep + 1 >= getTutorialSteps().length - 1;
    advanceTutorialStep({openModal:shouldOpenFinish});
  }

  function getTutorialJumpTarget(stepId){
    switch(stepId){
      case 'prefecture':
        return () => {
          if(S.currentMapMode !== 'japan') toggleMapMode('japan');
          resetMapView();
          closeModal();
        };
      case 'company':
      case 'stock':
        return () => {
          switchPanelView('market');
          closeModal();
        };
      case 'trade':
        return () => {
          switchPanelView('trade');
          closeModal();
        };
      case 'budget':
        return () => openAction('budget');
      case 'construction':
        return () => {
          switchPanelView('develop');
          closeModal();
        };
      case 'time':
        return () => {
          if(S.timeMode !== 'turn') setTimeMode('turn');
          closeModal();
        };
      default:
        return null;
    }
  }

  function startTutorialScenario(){
    if(!INITIAL_SNAPSHOT) return;
    applySaveData(JSON.parse(JSON.stringify(INITIAL_SNAPSHOT)));
    ensureTutorialState();
    S.tutorialActive = true;
    S.tutorialStep = 0;
    S.tutorialStartedWeek = S.totalWeeks || 0;
    S.started = false;
    S.currentMapMode = 'japan';
    S.currentPanel = 'market';
    S.japanFocusPrefecture = 'tokyo';
    S.money = Math.max(S.money, 700000);
    S.polCap = Math.max(S.polCap, 3200);
    S.majorEventCooldown = Math.max(S.majorEventCooldown || 0, 52);
    S.activeMajorEvent = null;
    S.timeMode = 'turn';
    S.turnStepWeeks = 1;
    S.turnProcessing = false;
    S.speed = 0;
    loadViewForMode('japan');
    clampMapView('japan');
    closeHomeScreen();
    updateUI();
    renderTutorialCard();
    openTutorialStepModal();
  }

  const rawUpdateHomeScreen = typeof updateHomeScreen === 'function' ? updateHomeScreen : null;
  if(rawUpdateHomeScreen){
    updateHomeScreen = function(message){
      ensureTutorialState();
      rawUpdateHomeScreen(message);
      ensureTutorialHomeButton();
    };
    window.updateHomeScreen = updateHomeScreen;
  }

  const rawUpdateUI = typeof updateUI === 'function' ? updateUI : null;
  if(rawUpdateUI){
    updateUI = function(){
      ensureTutorialState();
      rawUpdateUI();
      renderTutorialCard();
    };
    window.updateUI = updateUI;
  }

  const rawOpenPrefectureModal = typeof openPrefectureModal === 'function' ? openPrefectureModal : null;
  if(rawOpenPrefectureModal){
    openPrefectureModal = function(prefectureId, mode='prefecture'){
      rawOpenPrefectureModal(prefectureId, mode);
      handleTutorialAction('prefecture_open');
    };
    window.openPrefectureModal = openPrefectureModal;
  }

  const rawOpenCompanyDetail = typeof window.openCompanyDetail === 'function' ? window.openCompanyDetail : null;
  if(rawOpenCompanyDetail){
    window.openCompanyDetail = function(name){
      rawOpenCompanyDetail(name);
      handleTutorialAction('company_open');
    };
    openCompanyDetail = window.openCompanyDetail;
  }

  const rawBuyStock = typeof window.buyStock === 'function' ? window.buyStock : null;
  if(rawBuyStock){
    window.buyStock = function(name, qty){
      const before = S.ownedStocks[name] || 0;
      rawBuyStock(name, qty);
      if((S.ownedStocks[name] || 0) > before) handleTutorialAction('stock_bought');
    };
    buyStock = window.buyStock;
  }

  const rawStartImport = typeof window.startImport === 'function' ? window.startImport : null;
  if(rawStartImport){
    window.startImport = function(pi, res, pct){
      const before = (S.tradeDeals || []).length;
      rawStartImport(pi, res, pct);
      if((S.tradeDeals || []).length > before) handleTutorialAction('trade_started');
    };
    startImport = window.startImport;
  }

  const rawStartExport = typeof window.startExport === 'function' ? window.startExport : null;
  if(rawStartExport){
    window.startExport = function(pi, res, pct){
      const before = (S.tradeDeals || []).length;
      rawStartExport(pi, res, pct);
      if((S.tradeDeals || []).length > before) handleTutorialAction('trade_started');
    };
    startExport = window.startExport;
  }

  const rawApplyAnnualBudget = typeof window.applyAnnualBudget === 'function' ? window.applyAnnualBudget : null;
  if(rawApplyAnnualBudget){
    window.applyAnnualBudget = function(){
      const beforeYear = S.budgetSetYear;
      rawApplyAnnualBudget();
      if(S.budgetSetYear !== beforeYear && S.budgetSetYear === S.year) handleTutorialAction('budget_applied');
    };
    applyAnnualBudget = window.applyAnnualBudget;
  }

  const rawDevelopRegion = typeof window.developRegion === 'function' ? window.developRegion : null;
  if(rawDevelopRegion){
    window.developRegion = function(rk){
      const region = regions[rk];
      const before = region ? region.developed : 0;
      rawDevelopRegion(rk);
      if(region && region.developed > before) handleTutorialAction('construction_started');
    };
    developRegion = window.developRegion;
  }

  const rawPlacePublicWork = typeof placePublicWork === 'function' ? placePublicWork : null;
  if(rawPlacePublicWork){
    placePublicWork = function(workId, worldPos){
      const before = (S.publicWorks || []).length;
      const result = rawPlacePublicWork(workId, worldPos);
      if((S.publicWorks || []).length > before) handleTutorialAction('construction_started');
      return result;
    };
    window.placePublicWork = placePublicWork;
  }

  const rawGameTick = typeof gameTick === 'function' ? gameTick : null;
  if(rawGameTick){
    gameTick = function(options={}){
      const beforeWeeks = S.totalWeeks || 0;
      rawGameTick(options);
      if((S.totalWeeks || 0) > beforeWeeks) handleTutorialAction('time_advanced');
    };
    window.gameTick = gameTick;
  }

  window.startTutorialFromHome = function(){
    runWithViewLoader(getTutorialLocale().loader, () => {
      startTutorialScenario();
    });
  };

  window.tutorialOpenStep = function(){
    openTutorialStepModal();
  };

  window.tutorialJumpToContext = function(){
    const step = getCurrentTutorialStep();
    if(!step) return;
    const jump = getTutorialJumpTarget(step.id);
    if(jump) jump();
  };

  window.tutorialContinueStep = function(){
    const step = getCurrentTutorialStep();
    if(!step) return;
    if(step.id === 'wrapup'){
      completeTutorialFlow();
      return;
    }
    if(step.manual){
      advanceTutorialStep({openModal:true});
    }
  };

  window.tutorialSkipStep = function(){
    const step = getCurrentTutorialStep();
    if(!step || step.manual) return;
    addEvent(`${getTutorialLocale().stepSkipped}: ${step.title}`, '');
    advanceTutorialStep({openModal:false});
    closeModal();
  };

  window.tutorialExit = function(){
    ensureTutorialState();
    S.tutorialActive = false;
    renderTutorialCard();
    closeModal();
    addEvent(getTutorialLocale().exited, '');
  };

  ensureTutorialState();
  ensureTutorialHomeButton();
  renderTutorialCard();
})();

// ============================================================
// PREFECTURE ALLOCATION / POPULATION / PRODUCTIVITY LAYER
// ============================================================
(function(){
  const PREFECTURE_POP_WEIGHTS = {
    hokkaido:4.6,aomori:1.0,iwate:1.0,miyagi:2.1,akita:0.8,yamagata:0.9,fukushima:1.6,
    ibaraki:2.4,tochigi:1.8,gunma:1.7,saitama:5.8,chiba:5.2,tokyo:10.6,kanagawa:7.2,
    niigata:1.8,toyama:0.9,ishikawa:0.9,fukui:0.7,yamanashi:0.7,nagano:1.7,gifu:1.5,shizuoka:2.8,aichi:5.6,
    mie:1.4,shiga:1.1,kyoto:2.0,osaka:6.1,hyogo:4.4,nara:1.0,wakayama:0.8,
    tottori:0.5,shimane:0.5,okayama:1.5,hiroshima:2.2,yamaguchi:1.1,
    tokushima:0.6,kagawa:0.7,ehime:1.0,kochi:0.5,
    fukuoka:4.2,saga:0.7,nagasaki:1.0,kumamoto:1.4,oita:0.9,miyazaki:0.9,kagoshima:1.3,
    okinawa_pref:1.1,
  };
  const PREFECTURE_BASE_PRODUCTIVITY = {
    tokyo:12.5, kanagawa:9.8, osaka:9.1, aichi:8.6, saitama:7.4, chiba:7.1, hyogo:6.8, fukuoka:6.3,
    kyoto:5.7, shizuoka:5.4, ibaraki:4.9, tochigi:4.5, gunma:4.1, hokkaido:4.0, miyagi:4.4,
    niigata:3.6, okayama:3.4, hiroshima:3.8, gifu:3.2, mie:3.0, shiga:3.1, toyama:2.9, ishikawa:2.8,
    nagano:2.6, yamanashi:2.4, fukushima:2.5, yamagata:1.8, akita:1.6, aomori:1.5, iwate:1.7,
    nara:2.2, wakayama:1.6, tottori:1.2, shimane:1.1, yamaguchi:1.9, tokushima:1.4, kagawa:1.9,
    ehime:1.8, kochi:0.9, saga:1.5, nagasaki:1.8, kumamoto:2.3, oita:1.8, miyazaki:1.5, kagoshima:1.7,
    okinawa_pref:2.6,
  };
  const PRIORITY_LABELS = {
    0:'低',
    1:'標準',
    2:'高',
    3:'最優先',
  };

  function ownedCompanyRatio(company){
    return clamp(((S.ownedStocks?.[company.name] || 0) / Math.max(1, company.sharesOutstanding || 1)) * 100, 0, 100);
  }

  function getAllocationResourceKeys(){
    return Object.keys(resources).filter(resourceKey => companies.some(company => company.needs?.includes(resourceKey)));
  }

  const ORDINANCE_LABELS = {
    child:'子ども政策',
    elder:'高齢者支援',
    business:'企業誘致',
    housing:'住宅・移住',
  };

  const PREFECTURE_DEMOGRAPHIC_OVERRIDES = {
    tokyo:{children:0.108, seniors:0.236},
    kanagawa:{children:0.118, seniors:0.258},
    saitama:{children:0.122, seniors:0.267},
    chiba:{children:0.12, seniors:0.272},
    aichi:{children:0.122, seniors:0.274},
    osaka:{children:0.11, seniors:0.304},
    hyogo:{children:0.114, seniors:0.314},
    kyoto:{children:0.109, seniors:0.318},
    fukuoka:{children:0.121, seniors:0.286},
    miyagi:{children:0.119, seniors:0.296},
    hokkaido:{children:0.108, seniors:0.326},
    aomori:{children:0.1, seniors:0.364},
    akita:{children:0.094, seniors:0.394},
    yamagata:{children:0.098, seniors:0.374},
    shimane:{children:0.097, seniors:0.388},
    kochi:{children:0.094, seniors:0.392},
    kagoshima:{children:0.105, seniors:0.351},
    okinawa_pref:{children:0.166, seniors:0.238},
  };

  function getPrefectureDemographicSeed(prefectureId){
    const override = PREFECTURE_DEMOGRAPHIC_OVERRIDES[prefectureId];
    if(override){
      return {
        children:override.children,
        seniors:override.seniors,
        working:Math.max(0.42, 1 - override.children - override.seniors),
      };
    }
    const productivity = PREFECTURE_BASE_PRODUCTIVITY[prefectureId] ?? 2;
    const seniors = clamp(0.29 + (2.2 - productivity) * 0.012, 0.23, 0.38);
    const children = clamp(0.116 + Math.max(0, 2.8 - seniors * 6.5) * 0.002, 0.098, 0.15);
    return {
      children,
      seniors,
      working:Math.max(0.44, 1 - children - seniors),
    };
  }

  function createDefaultOrdinances(){
    return {child:1, elder:1, business:1, housing:1};
  }

  function syncPrefectureDemographicShape(prefectureId, stat){
    const population = Math.max(12, Number(stat.population) || 12);
    if(typeof stat.birthRate !== 'number') stat.birthRate = S.birthRate || 7.4;
    if(typeof stat.births !== 'number') stat.births = 0;
    if(typeof stat.deaths !== 'number') stat.deaths = 0;
    if(typeof stat.netMigration !== 'number') stat.netMigration = 0;
    if(typeof stat.ordinanceMode !== 'string') stat.ordinanceMode = 'auto';
    if(!stat.ordinances || typeof stat.ordinances !== 'object') stat.ordinances = createDefaultOrdinances();
    Object.keys(ORDINANCE_LABELS).forEach(key => {
      if(typeof stat.ordinances[key] !== 'number') stat.ordinances[key] = 1;
      stat.ordinances[key] = clamp(Number(stat.ordinances[key]), 0, 3);
    });
    if(typeof stat.children !== 'number' || typeof stat.workingAge !== 'number' || typeof stat.seniors !== 'number'){
      const seed = getPrefectureDemographicSeed(prefectureId);
      stat.children = population * seed.children;
      stat.workingAge = population * seed.working;
      stat.seniors = population * seed.seniors;
    }
    const total = Math.max(1, (Number(stat.children) || 0) + (Number(stat.workingAge) || 0) + (Number(stat.seniors) || 0));
    const scale = population / total;
    stat.children = Math.max(0.5, stat.children * scale);
    stat.workingAge = Math.max(1, stat.workingAge * scale);
    stat.seniors = Math.max(0.5, stat.seniors * scale);
    stat.population = stat.children + stat.workingAge + stat.seniors;
  }

  function getNationalDemographicSummary(){
    ensureAllocationState();
    const summary = prefectures.reduce((acc, prefecture) => {
      const stat = getPrefectureStat(prefecture.id);
      if(!stat) return acc;
      acc.children += stat.children || 0;
      acc.workingAge += stat.workingAge || 0;
      acc.seniors += stat.seniors || 0;
      acc.population += stat.population || 0;
      acc.birthRateWeighted += (stat.birthRate || 0) * (stat.population || 0);
      return acc;
    }, {children:0, workingAge:0, seniors:0, population:0, birthRateWeighted:0});
    summary.birthRate = summary.population > 0 ? summary.birthRateWeighted / summary.population : (S.birthRate || 0);
    summary.childShare = summary.population > 0 ? summary.children / summary.population : 0;
    summary.workingShare = summary.population > 0 ? summary.workingAge / summary.population : 0;
    summary.seniorShare = summary.population > 0 ? summary.seniors / summary.population : 0;
    summary.oldAgeDependency = summary.workingAge > 0 ? summary.seniors / summary.workingAge : 0;
    return summary;
  }

  function buildPopulationPyramidSvg(summary){
    const groups = [
      {label:'高齢', value:summary.seniors, color:'#ff8a80'},
      {label:'生産年齢', value:summary.workingAge, color:'#4fc3f7'},
      {label:'子ども', value:summary.children, color:'#81c784'},
    ];
    const max = Math.max(1, ...groups.map(group => group.value));
    const rows = groups.map((group, index) => {
      const y = 26 + index * 44;
      const leftWidth = (group.value / max) * 114 * 0.49;
      const rightWidth = (group.value / max) * 114 * 0.51;
      return `
        <rect x="${160 - leftWidth}" y="${y}" width="${leftWidth}" height="24" rx="8" fill="${group.color}" opacity=".82"></rect>
        <rect x="160" y="${y}" width="${rightWidth}" height="24" rx="8" fill="${group.color}" opacity=".96"></rect>
        <text x="160" y="${y - 6}" text-anchor="middle" fill="#dbe8f6" font-size="12" font-weight="700">${group.label}</text>
      `;
    }).join('');
    return `<svg viewBox="0 0 320 172" style="width:100%;height:200px">
      <line x1="160" y1="14" x2="160" y2="156" stroke="rgba(255,255,255,.18)" stroke-width="2"></line>
      <text x="96" y="166" text-anchor="middle" fill="#9ab0c9" font-size="11">男性寄り</text>
      <text x="224" y="166" text-anchor="middle" fill="#9ab0c9" font-size="11">女性寄り</text>
      ${rows}
    </svg>`;
  }
  window.getNationalDemographicSummary = getNationalDemographicSummary;
  window.buildPopulationPyramidSvg = buildPopulationPyramidSvg;

  function movePopulationBetweenPrefectures(sourceStat, targetStat, volume){
    if(!sourceStat || !targetStat) return;
    const movable = clamp(Number(volume) || 0, 0, Math.max(0, (sourceStat.population || 0) - 12));
    if(movable <= 0) return;
    syncPrefectureDemographicShape('', sourceStat);
    syncPrefectureDemographicShape('', targetStat);
    const sourceTotal = Math.max(1, sourceStat.population || 1);
    const childMove = movable * ((sourceStat.children || 0) / sourceTotal);
    const workerMove = movable * ((sourceStat.workingAge || 0) / sourceTotal);
    const seniorMove = movable * ((sourceStat.seniors || 0) / sourceTotal);
    sourceStat.children = Math.max(0.5, sourceStat.children - childMove);
    sourceStat.workingAge = Math.max(1, sourceStat.workingAge - workerMove);
    sourceStat.seniors = Math.max(0.5, sourceStat.seniors - seniorMove);
    targetStat.children += childMove;
    targetStat.workingAge += workerMove;
    targetStat.seniors += seniorMove;
    sourceStat.population = sourceStat.children + sourceStat.workingAge + sourceStat.seniors;
    targetStat.population = targetStat.children + targetStat.workingAge + targetStat.seniors;
    sourceStat.netMigration = (sourceStat.netMigration || 0) - movable;
    targetStat.netMigration = (targetStat.netMigration || 0) + movable;
  }

  function autoTunePrefectureOrdinances(prefectureId, stat){
    const population = Math.max(1, stat.population || 1);
    const childrenShare = (stat.children || 0) / population;
    const seniorShare = (stat.seniors || 0) / population;
    stat.ordinances.child = clamp(Math.round(1 + (0.122 - childrenShare) * 24 + Math.max(0, 55 - (stat.appeal || 50)) * 0.012), 0, 3);
    stat.ordinances.elder = clamp(Math.round(1 + Math.max(0, seniorShare - 0.29) * 18 + Math.max(0, 48 - (stat.wealth || 0)) * 0.012), 0, 3);
    stat.ordinances.business = clamp(Math.round(1 + Math.max(0, 3.8 - (stat.productivity || 0)) * 0.18 + Math.max(0, 52 - (stat.appeal || 50)) * 0.01), 0, 3);
    stat.ordinances.housing = clamp(Math.round(1 + Math.max(0, 54 - (stat.appeal || 50)) * 0.016 + Math.max(0, 1 - getPrefectureTransportConnectivity(prefectureId)) * 0.4), 0, 3);
  }

  function applyPrefectureDemographicsTick(){
    ensureAllocationState();
    const totals = {socialSecurity:0, healthcare:0, childcare:0, administration:0, births:0, deaths:0, population:0, birthRateWeighted:0};
    prefectures.forEach(prefecture => {
      const stat = getPrefectureStat(prefecture.id);
      if(!stat) return;
      syncPrefectureDemographicShape(prefecture.id, stat);
      if(stat.ordinanceMode !== 'manual') autoTunePrefectureOrdinances(prefecture.id, stat);
      const population = Math.max(1, stat.population || 1);
      const childLevel = stat.ordinances.child || 0;
      const elderLevel = stat.ordinances.elder || 0;
      const businessLevel = stat.ordinances.business || 0;
      const housingLevel = stat.ordinances.housing || 0;
      const childShare = (stat.children || 0) / population;
      const seniorShare = (stat.seniors || 0) / population;
      const transit = getPrefectureTransportConnectivity(prefecture.id);
      const region = regions[prefecture.region];
      const disasterPenalty = region?.disaster ? (region.disaster.severity || 0) * 0.22 : 0;
      const economyBoost = Number(stat.economyBoost || 0);
      const populationBoost = Number(stat.populationBoost || 0);
      const prosperity = clamp(
        1
        + S.businessCycle * 0.38
        + Math.max(0, stat.productivity || 0) / 140
        + Math.max(0, (stat.appeal || 50) - 50) / 190
        + transit * 0.016
        + economyBoost * 0.014
        + populationBoost * 0.009
        - Math.max(0, S.socialUnrest - 24) * 0.008
        - Math.max(0, S.unemployment - 4.6) * 0.028
        - disasterPenalty,
        0.72, 1.46
      );
      const birthRate = clamp(
        5.4
        + childLevel * 0.82
        + housingLevel * 0.58
        + businessLevel * 0.16
        + Math.max(0, prosperity - 1) * 2.6
        + populationBoost * 0.22
        + economyBoost * 0.05
        - seniorShare * 3.8
        + (prefecture.id === 'okinawa_pref' ? 0.85 : 0),
        3.1, 11.9
      );
      const births = population * birthRate / 52000;
      const ageToWorking = stat.children * 0.00118;
      const workToSenior = stat.workingAge * 0.00054;
      const childDeaths = stat.children * (0.00003 + disasterPenalty * 0.0005);
      const workingDeaths = stat.workingAge * (0.00005 + disasterPenalty * 0.00072);
      const seniorDeaths = stat.seniors * (0.00082 + Math.max(0, seniorShare - 0.31) * 0.0004 + disasterPenalty * 0.0013 - elderLevel * 0.00006);
      stat.children = Math.max(0.5, stat.children + births - ageToWorking - childDeaths);
      stat.workingAge = Math.max(1, stat.workingAge + ageToWorking - workToSenior - workingDeaths);
      stat.seniors = Math.max(0.5, stat.seniors + workToSenior - seniorDeaths);
      stat.population = stat.children + stat.workingAge + stat.seniors;
      stat.birthRate = birthRate;
      stat.births = births;
      stat.deaths = childDeaths + workingDeaths + seniorDeaths;
      stat.productivity = clamp(
        stat.productivity
        + (businessLevel - 1) * 0.03
        + (housingLevel - 1) * 0.012
        + economyBoost * 0.028
        + populationBoost * 0.006
        - Math.max(0, seniorShare - 0.32) * 0.05
        + Math.max(0, (stat.foodAdequacy || 1) - 1) * 0.07
        - Math.max(0, 1 - (stat.foodAdequacy || 1)) * 0.1,
        -35, 90
      );
      stat.appeal = clamp(
        (stat.appeal || 50)
        + (housingLevel - 1) * 0.08
        + (childLevel - 1) * 0.04
        + (elderLevel - 1) * 0.03
        + populationBoost * 0.06
        + economyBoost * 0.035
        + Math.max(0, prosperity - 1) * 0.26
        - disasterPenalty * 3.2,
        0, 100
      );
      totals.population += stat.population;
      totals.births += births;
      totals.deaths += stat.deaths;
      totals.birthRateWeighted += birthRate * stat.population;
      totals.socialSecurity += stat.seniors * (0.088 + elderLevel * 0.015);
      totals.healthcare += population * 0.013 + stat.seniors * (0.026 + elderLevel * 0.006);
      totals.childcare += stat.children * (0.036 + childLevel * 0.016);
      totals.administration += population * 0.003 + (businessLevel + housingLevel) * 1.5;
    });
    S.pop = totals.population;
    S.birthRate = totals.population > 0 ? totals.birthRateWeighted / totals.population : (S.birthRate || 7.4);
    S.money -= (totals.socialSecurity + totals.healthcare + totals.childcare + totals.administration);
    S.weeklyBreakdown.socialSecurity = (S.weeklyBreakdown.socialSecurity || 0) + totals.socialSecurity;
    S.weeklyBreakdown.healthcare = (S.weeklyBreakdown.healthcare || 0) + totals.healthcare;
    S.weeklyBreakdown.childcare = (S.weeklyBreakdown.childcare || 0) + totals.childcare;
    S.weeklyBreakdown.administration = (S.weeklyBreakdown.administration || 0) + totals.administration;
  }

  function ensureAllocationState(){
    if(typeof S.resourceAllocationMode !== 'string') S.resourceAllocationMode = 'auto';
    if(typeof S.foodAllocationMode !== 'string') S.foodAllocationMode = 'auto';
    if(!S.companyResourcePlans || typeof S.companyResourcePlans !== 'object') S.companyResourcePlans = {};
    if(!S.companyResourceDeliveries || typeof S.companyResourceDeliveries !== 'object') S.companyResourceDeliveries = {};
    if(!S.prefectureStats || typeof S.prefectureStats !== 'object' || !Object.keys(S.prefectureStats).length){
      S.prefectureStats = {};
      const weightSum = prefectures.reduce((sum, prefecture) => sum + (PREFECTURE_POP_WEIGHTS[prefecture.id] || 1), 0);
      prefectures.forEach(prefecture => {
        const weight = PREFECTURE_POP_WEIGHTS[prefecture.id] || 1;
        S.prefectureStats[prefecture.id] = {
          population:Math.max(18, S.pop * (weight / weightSum)),
          productivity:PREFECTURE_BASE_PRODUCTIVITY[prefecture.id] ?? rand(-1.8, 1.8),
          foodPriority:1,
          foodAdequacy:1,
          foodReceived:0,
          lastFoodGap:0,
          appeal:50,
          wealth:0,
          economyBoost:0,
          populationBoost:0,
          ordinanceMode:'auto',
          ordinances:createDefaultOrdinances(),
        };
      });
    }
    prefectures.forEach(prefecture => {
      if(!S.prefectureStats[prefecture.id]){
        S.prefectureStats[prefecture.id] = {population:Math.max(18, S.pop / prefectures.length), productivity:PREFECTURE_BASE_PRODUCTIVITY[prefecture.id] ?? 0, foodPriority:1, foodAdequacy:1, foodReceived:0, lastFoodGap:0, appeal:50, wealth:0, economyBoost:0, populationBoost:0, ordinanceMode:'auto', ordinances:createDefaultOrdinances()};
      }
      const stat = S.prefectureStats[prefecture.id];
      if(typeof stat.population !== 'number') stat.population = Math.max(18, S.pop / prefectures.length);
      if(typeof stat.productivity !== 'number') stat.productivity = PREFECTURE_BASE_PRODUCTIVITY[prefecture.id] ?? 0;
      if(typeof stat.foodPriority !== 'number') stat.foodPriority = 1;
      if(typeof stat.foodAdequacy !== 'number') stat.foodAdequacy = 1;
      if(typeof stat.foodReceived !== 'number') stat.foodReceived = 0;
      if(typeof stat.lastFoodGap !== 'number') stat.lastFoodGap = 0;
      if(typeof stat.appeal !== 'number') stat.appeal = 50;
      if(typeof stat.wealth !== 'number') stat.wealth = 0;
      if(typeof stat.economyBoost !== 'number') stat.economyBoost = 0;
      if(typeof stat.populationBoost !== 'number') stat.populationBoost = 0;
      syncPrefectureDemographicShape(prefecture.id, stat);
    });
    if(typeof S.resourceAllocationFocus !== 'string' || !resources[S.resourceAllocationFocus] || !companies.some(company => company.needs?.includes(S.resourceAllocationFocus))){
      S.resourceAllocationFocus = getAllocationResourceKeys()[0] || 'iron';
    }
    if(typeof S.prefectureModalState !== 'object' || !S.prefectureModalState) S.prefectureModalState = {prefectureId:S.japanFocusPrefecture || 'tokyo', mode:'prefecture'};
  }

  function getPrefectureStat(prefectureId){
    ensureAllocationState();
    return S.prefectureStats[prefectureId] || null;
  }

  function getAveragePrefecturePopulation(){
    ensureAllocationState();
    const values = Object.values(S.prefectureStats);
    return values.length ? values.reduce((sum, stat) => sum + stat.population, 0) / values.length : Math.max(1, S.pop / Math.max(1, prefectures.length));
  }

  function getAveragePrefectureProductivity(){
    ensureAllocationState();
    const values = Object.values(S.prefectureStats);
    return values.length ? values.reduce((sum, stat) => sum + stat.productivity, 0) / values.length : 0;
  }

  function getAveragePrefectureAppeal(){
    ensureAllocationState();
    const values = Object.values(S.prefectureStats);
    return values.length ? values.reduce((sum, stat) => sum + (stat.appeal || 50), 0) / values.length : 50;
  }

  function getPrefectureTransportConnectivity(prefectureId){
    return getPrefectureTransportRoutes(prefectureId, true).reduce((sum, route) => {
      const config = transportNetworkCatalog[route.type];
      if(!config) return sum;
      return sum + 0.45 + config.productivity * 0.08 + config.appeal * 0.06;
    }, 0);
  }

  function updatePrefectureAppealScores(){
    ensureAllocationState();
    prefectures.forEach(prefecture => {
      const stat = getPrefectureStat(prefecture.id);
      const region = regions[prefecture.region];
      const localCompanies = getPrefectureCompanies(prefecture.id);
      const localWealth = getPrefectureEconomicMass(prefecture.id);
      const publicBoost = (S.publicWorks || []).filter(work => work.status === 'active' && work.region === prefecture.region).length;
      const transportBoost = getPrefectureTransportConnectivity(prefecture.id);
      const disasterPenalty = region?.disaster ? 16 + (region.disaster.severity || 0) * 12 : 0;
      const economyBoost = Number(stat.economyBoost || 0);
      const populationBoost = Number(stat.populationBoost || 0);
      stat.wealth = clamp(
        localWealth * 0.9
        + localCompanies.length * 2.2
        + (region?.developed || 0) * 0.7
        + economyBoost * 1.6
        + populationBoost * 0.45,
        0,
        100
      );
      stat.appeal = clamp(
        42
        + stat.productivity * 0.72
        + (stat.foodAdequacy - 1) * 32
        + stat.wealth * 0.32
        + transportBoost * 1.7
        + publicBoost * 2.6
        + economyBoost * 0.7
        + populationBoost * 0.94
        - disasterPenalty,
        0,
        100
      );
    });
  }

  function applyConnectedPopulationMigration(){
    ensureAllocationState();
    const activeRoutes = (S.transportNetworks || []).filter(route => route.status === 'active');
    activeRoutes.forEach(route => {
      const fromStat = getPrefectureStat(route.fromPrefectureId);
      const toStat = getPrefectureStat(route.toPrefectureId);
      const config = transportNetworkCatalog[route.type];
      if(!fromStat || !toStat || !config) return;
      const fromAppeal = fromStat.appeal || 50;
      const toAppeal = toStat.appeal || 50;
      const diff = Math.abs(fromAppeal - toAppeal);
      if(diff < 2.4) return;
      const moveFrom = fromAppeal >= toAppeal ? route.toPrefectureId : route.fromPrefectureId;
      const moveTo = fromAppeal >= toAppeal ? route.fromPrefectureId : route.toPrefectureId;
      const source = getPrefectureStat(moveFrom);
      const target = getPrefectureStat(moveTo);
      if(!source || !target || source.population <= 18) return;
      const volume = clamp(diff * 0.045 * config.migration, 0.18, Math.min(source.population * 0.008, 2.8));
      movePopulationBetweenPrefectures(source, target, volume);
      target.appeal = clamp((target.appeal || 50) + volume * 0.12, 0, 100);
      source.appeal = clamp((source.appeal || 50) - volume * 0.08, 0, 100);
    });
    S.pop = prefectures.reduce((sum, prefecture) => sum + (getPrefectureStat(prefecture.id)?.population || 0), 0);
  }

  function arePrefecturesTransitConnected(fromId, toId){
    if(fromId === toId) return true;
    const activeRoutes = (S.transportNetworks || []).filter(route => route.status === 'active');
    const seen = new Set([fromId]);
    const queue = [fromId];
    while(queue.length){
      const current = queue.shift();
      for(const route of activeRoutes){
        const next = route.fromPrefectureId === current ? route.toPrefectureId : route.toPrefectureId === current ? route.fromPrefectureId : null;
        if(!next || seen.has(next)) continue;
        if(next === toId) return true;
        seen.add(next);
        queue.push(next);
      }
    }
    return false;
  }

  function getCompanyResourcePriority(companyName, resourceKey){
    ensureAllocationState();
    return clamp(Number(S.companyResourcePlans?.[companyName]?.[resourceKey] ?? 1), 0, 3);
  }

  function setCompanyResourcePriority(companyName, resourceKey, value){
    ensureAllocationState();
    if(!S.companyResourcePlans[companyName]) S.companyResourcePlans[companyName] = {};
    S.companyResourcePlans[companyName][resourceKey] = clamp(Number(value), 0, 3);
  }

  function setPrefectureFoodPriority(prefectureId, value){
    const stat = getPrefectureStat(prefectureId);
    if(!stat) return;
    stat.foodPriority = clamp(Number(value), 0, 3);
  }

  function getOverallFoodPool(){
    const rice = resources.rice ? resources.rice.amount / Math.max(1, resources.rice.max) : 0.6;
    const fish = resources.fish ? resources.fish.amount / Math.max(1, resources.fish.max) : 0.6;
    return clamp(rice * 0.7 + fish * 0.3, 0.24, 1.22);
  }

  function allocateFoodForTick(){
    ensureAllocationState();
    const overallFoodPool = getOverallFoodPool();
    const entries = prefectures.map(prefecture => {
      const stat = getPrefectureStat(prefecture.id);
      const region = regions[prefecture.region];
      const localCompanies = getPrefectureCompanies(prefecture.id);
      const demand = Math.max(8, stat.population * (1 + Math.max(0, stat.productivity) / 260));
      const priorityHint = 0.86 + stat.foodPriority * 0.18;
      const autoWeight = demand * priorityHint * (1 + Math.max(0, -stat.productivity) / 120 + localCompanies.length * 0.018 + ((region?.developed || 0) / 120));
      const manualWeight = demand * (0.64 + stat.foodPriority * 0.5);
      return {prefecture, stat, demand, weight:S.foodAllocationMode === 'manual' ? manualWeight : autoWeight};
    });
    const totalDemand = entries.reduce((sum, entry) => sum + entry.demand, 0) || 1;
    const totalWeight = entries.reduce((sum, entry) => sum + entry.weight, 0) || 1;
    entries.forEach(entry => {
      const fairShare = entry.demand / totalDemand;
      const allocatedShare = entry.weight / totalWeight;
      const adequacy = clamp(overallFoodPool * (allocatedShare / Math.max(0.0001, fairShare)), 0.45, 1.35);
      const productivityTarget = clamp((adequacy - 1) * 26 + (regions[entry.prefecture.region]?.developed || 0) * 0.45 + getPrefectureCompanies(entry.prefecture.id).length * 0.35, -24, 42);
      entry.stat.foodAdequacy = adequacy;
      entry.stat.foodReceived = adequacy * entry.demand;
      entry.stat.lastFoodGap = adequacy - 1;
      entry.stat.productivity = clamp(entry.stat.productivity + (productivityTarget - entry.stat.productivity) * 0.18, -35, 60);
    });

    updatePrefectureAppealScores();
    const targetNationalPopulation = Math.max(100, S.pop);
    entries.forEach(entry => {
      const naturalGrowth = (entry.stat.foodAdequacy - 1) * 0.18 + entry.stat.productivity * 0.004 + ((entry.stat.appeal || 50) - 50) * 0.003;
      entry.stat.population = Math.max(12, entry.stat.population * (1 + naturalGrowth * 0.01));
    });
    const popSum = entries.reduce((sum, entry) => sum + entry.stat.population, 0) || 1;
    const popScale = targetNationalPopulation / popSum;
    entries.forEach(entry => {
      entry.stat.population = Math.max(12, entry.stat.population * popScale);
    });
    S.pop = entries.reduce((sum, entry) => sum + entry.stat.population, 0);
    return {
      stressed:entries.filter(entry => entry.stat.foodAdequacy < 0.92).length,
      opsCost:Math.round(entries.length * 5 + Math.max(0, 1 - overallFoodPool) * 240),
      overallFoodPool,
    };
  }

  function allocateCompanyResourcesForTick(){
    ensureAllocationState();
    S.companyResourceDeliveries = {};
    const opsTouched = new Set();
    Object.keys(resources).forEach(resourceKey => {
      const targetCompanies = companies.filter(company => company.needs?.includes(resourceKey));
      if(!targetCompanies.length) return;
      const resource = resources[resourceKey];
      const supplyLevel = resource && resource.max ? clamp(resource.amount / Math.max(1, resource.max), 0.24, 1.2) : 0.88;
      const weighted = targetCompanies.map(company => {
        const stat = getPrefectureStat(company.prefecture);
        const priority = getCompanyResourcePriority(company.name, resourceKey);
        const baseWeight = 1 + (company.favorability || 50) / 220 + (company.techDepth || 20) / 260 + ownedCompanyRatio(company) / 260;
        const prefectureFactor = stat ? 1 + Math.max(-0.08, stat.productivity / 220) + Math.max(-0.06, stat.foodAdequacy - 1) : 1;
        const autoPriority = 0.88 + priority * 0.14;
        const manualPriority = 0.62 + priority * 0.48;
        return {
          company,
          weight:baseWeight * prefectureFactor * (S.resourceAllocationMode === 'manual' ? manualPriority : autoPriority),
        };
      });
      const totalWeight = weighted.reduce((sum, entry) => sum + entry.weight, 0) || 1;
      const fairShare = 1 / weighted.length;
      weighted.forEach(entry => {
        const share = entry.weight / totalWeight;
        const coverage = clamp(supplyLevel * (share / fairShare), 0.35, 1.28);
        if(!S.companyResourceDeliveries[entry.company.name]) S.companyResourceDeliveries[entry.company.name] = {};
        S.companyResourceDeliveries[entry.company.name][resourceKey] = coverage;
        opsTouched.add(entry.company.name);
      });
    });
    return {
      opsCost:Math.round(opsTouched.size * 4 + Object.keys(S.companyResourceDeliveries).length * 2),
    };
  }

  function applyPrefectureImpactToCompanies(){
    ensureAllocationState();
    const avgPopulation = getAveragePrefecturePopulation();
    companies.forEach(company => {
      const stat = getPrefectureStat(company.prefecture);
      const deliveries = S.companyResourceDeliveries[company.name] || {};
      const deliveryValues = company.needs?.length ? company.needs.map(resourceKey => deliveries[resourceKey] ?? 0.92) : [1];
      const deliveryAvg = deliveryValues.reduce((sum, value) => sum + value, 0) / Math.max(1, deliveryValues.length);
      const supplyCompletion = company.needs?.length
        ? company.needs.filter(resourceKey => (deliveries[resourceKey] ?? 0.92) >= 0.96).length / company.needs.length
        : 1;
      const populationPressure = stat ? clamp(stat.population / Math.max(1, avgPopulation), 0.72, 2.2) : 1;
      const workingShare = stat ? clamp((stat.workingAge || stat.population || 1) / Math.max(1, stat.population || 1), 0.45, 0.75) : 0.6;
      const transportConnectivity = stat ? getPrefectureTransportConnectivity(company.prefecture) : 0;
      const economyBoost = stat ? Number(stat.economyBoost || 0) : 0;
      const populationBoost = stat ? Number(stat.populationBoost || 0) : 0;
      const localDemandFactor = stat ? clamp(0.97 + (populationPressure - 1) * 0.14 + stat.productivity / 165 + (stat.foodAdequacy - 1) * 0.15 + ((stat.appeal || 50) - 50) / 360 + (workingShare - 0.6) * 0.3 + economyBoost * 0.006 + populationBoost * 0.004, 0.86, 1.42) : 1;
      const logisticsFactor = clamp(0.93 + supplyCompletion * 0.09 + (deliveryAvg - 1) * 0.22 + transportConnectivity * 0.014, 0.84, 1.19);
      const productivityFactor = stat ? clamp(0.98 + stat.productivity / 230 + (stat.wealth || 0) / 720 + (workingShare - 0.6) * 0.16 + economyBoost * 0.005, 0.9, 1.26) : 1;
      const finalRevenueFactor = clamp(localDemandFactor * logisticsFactor * productivityFactor, 0.82, 1.42);
      const revenueGrowth = finalRevenueFactor - 1;
      const finalPriceFactor = clamp(0.992 + revenueGrowth * 0.24 + ((stat?.appeal || 50) - 50) / 1300 + (Math.random() - 0.5) * 0.014, 0.96, 1.05);
      const revenueLerpFactor = 1 + revenueGrowth * 0.18;
      company.revenue = Math.max(120, company.revenue * clamp(revenueLerpFactor, 0.95, 1.06));
      company.price = Math.max(getCompanyDynamicFloor(company), Math.round(company.price * finalPriceFactor));
      if(Array.isArray(company.revenueHist) && company.revenueHist.length) company.revenueHist[company.revenueHist.length - 1] = Math.round(company.revenue);
      if(Array.isArray(company.hist) && company.hist.length) company.hist[company.hist.length - 1] = company.price;
    });
  }

  function getPrefectureFoodPriorityButtons(prefectureId){
    const stat = getPrefectureStat(prefectureId);
    const active = stat ? stat.foodPriority : 1;
    return [0,1,2,3].map(value => `<button class="btn btn-sm ${active===value?'active':''}" onclick="setPrefectureFoodPriorityLevel('${prefectureId}', ${value})">${PRIORITY_LABELS[value]}</button>`).join('');
  }

  function getCompanyPriorityButtons(companyName, resourceKey){
    const active = getCompanyResourcePriority(companyName, resourceKey);
    return [0,1,2,3].map(value => `<button class="btn btn-sm ${active===value?'active':''}" onclick="setCompanyResourcePriorityLevel('${companyName}', '${resourceKey}', ${value})">${value}</button>`).join('');
  }

  function renderAllocationSummarySection(){
    const avgProductivity = getAveragePrefectureProductivity();
    const avgPopulation = getAveragePrefecturePopulation();
    const avgAppeal = getAveragePrefectureAppeal();
    const foodPool = getOverallFoodPool();
    const demo = getNationalDemographicSummary();
    return `<div class="section">
      <h3>🚚 配分管制</h3>
      <div class="save-actions" style="margin-bottom:8px">
        <button class="btn btn-sm ${S.resourceAllocationMode==='auto'?'btn-blue active':''}" onclick="setResourceAllocationMode('auto')">企業資源 自動</button>
        <button class="btn btn-sm ${S.resourceAllocationMode==='manual'?'active':''}" onclick="setResourceAllocationMode('manual')">企業資源 手動</button>
        <button class="btn btn-sm ${S.foodAllocationMode==='auto'?'btn-green active':''}" onclick="setFoodAllocationMode('auto')">食料配分 自動</button>
        <button class="btn btn-sm ${S.foodAllocationMode==='manual'?'active':''}" onclick="setFoodAllocationMode('manual')">食料配分 手動</button>
      </div>
      <div class="detail-grid">
        <div class="dg-item"><div class="dg-label">平均県人口</div><div class="dg-value">${avgPopulation.toFixed(0)}万</div></div>
        <div class="dg-item"><div class="dg-label">平均生産性</div><div class="dg-value">${avgProductivity >= 0 ? '+' : ''}${avgProductivity.toFixed(1)}%</div></div>
        <div class="dg-item"><div class="dg-label">食料充足</div><div class="dg-value">${(foodPool * 100).toFixed(0)}%</div></div>
        <div class="dg-item"><div class="dg-label">平均魅力度</div><div class="dg-value">${avgAppeal.toFixed(1)}</div></div>
        <div class="dg-item"><div class="dg-label">焦点県</div><div class="dg-value">${prefectureMap[S.japanFocusPrefecture]?.name || '未選択'}</div></div>
        <div class="dg-item"><div class="dg-label">出生率</div><div class="dg-value">${demo.birthRate.toFixed(2)}</div></div>
        <div class="dg-item"><div class="dg-label">高齢化率</div><div class="dg-value">${(demo.seniorShare * 100).toFixed(1)}%</div></div>
      </div>
      <div class="compact-note" style="margin-top:8px">自動モードは全体最適を優先し、手動モードではあなたの優先度が強く反映されます。県人口・年齢構成・生産性・魅力度・交通接続が高い県ほど、その県の企業の売上と収益性が伸びやすく、好景気ほど出生率も上向きます。</div>
    </div>`;
  }

  function getPrefectureOrdinanceButtons(prefectureId, key){
    const stat = getPrefectureStat(prefectureId);
    const active = clamp(Number(stat?.ordinances?.[key] ?? 1), 0, 3);
    const disabled = stat?.ordinanceMode !== 'manual';
    return [0,1,2,3].map(value => `<button class="btn btn-sm ${active===value?'active':''}" ${disabled ? 'disabled' : ''} onclick="setPrefectureOrdinanceLevel('${prefectureId}', '${key}', ${value})">${value}</button>`).join('');
  }

  function renderCompanyAllocationSection(){
    const allocationKeys = getAllocationResourceKeys();
    const selectedKey = allocationKeys.includes(S.resourceAllocationFocus) ? S.resourceAllocationFocus : allocationKeys[0];
    const resource = resources[selectedKey];
    const targetCompanies = companies.filter(company => company.needs?.includes(selectedKey)).sort((a,b) => {
      const aRatio = (S.companyResourceDeliveries?.[a.name]?.[selectedKey] ?? 1);
      const bRatio = (S.companyResourceDeliveries?.[b.name]?.[selectedKey] ?? 1);
      return aRatio - bRatio;
    });
    return `<div class="section">
      <h3>🏭 企業への資源配分</h3>
      <div class="save-actions" style="margin-bottom:8px">
        ${allocationKeys.map(resourceKey => `<button class="btn btn-sm ${selectedKey===resourceKey?'active':''}" onclick="setResourceAllocationFocus('${resourceKey}')">${resources[resourceKey]?.name || resourceKey}</button>`).join('')}
      </div>
      <div class="compact-note" style="margin-bottom:8px">${resource?.name || selectedKey} を必要とする企業へ優先度を付けられます。いまの在庫は ${formatAmount(resource?.amount || 0)} ${resource?.unit || ''} です。</div>
      ${targetCompanies.map(company => {
        const stat = getPrefectureStat(company.prefecture);
        const delivery = S.companyResourceDeliveries?.[company.name]?.[selectedKey] ?? 1;
        return `<div class="company" style="margin-top:8px;border-left-color:${sectors[company.sector]?.color || '#90caf9'}">
          <div class="c-left" style="display:flex;flex-direction:column;align-items:flex-start">
            <span class="c-name">${sectors[company.sector]?.icon || '🏢'}${company.name}</span>
            <span class="compact-note">${prefectureMap[company.prefecture]?.name || company.region} / 生産性 ${stat ? (stat.productivity >= 0 ? '+' : '') + stat.productivity.toFixed(1) + '%' : '--'} / 配分 ${Math.round(delivery * 100)}%</span>
          </div>
          <div class="c-right" style="display:flex;gap:4px;flex-wrap:wrap;justify-content:flex-end">
            ${getCompanyPriorityButtons(company.name, selectedKey)}
          </div>
        </div>`;
      }).join('') || `<div class="compact-note">この資源を直接必要とする企業はありません。</div>`}
    </div>`;
  }

  function renderFoodAllocationSection(){
    const regionBlocks = Object.entries(prefecturesByRegion).map(([regionId, list]) => {
      const region = regions[regionId];
      const rows = list.map(prefecture => {
        const stat = getPrefectureStat(prefecture.id);
        const transport = getPrefectureTransportRoutes(prefecture.id, true).length;
        return `<div class="trade-item" style="align-items:flex-start">
          <span style="display:flex;flex-direction:column;align-items:flex-start;gap:2px">
            <strong>${prefecture.name}</strong>
            <span class="compact-note">人口 ${stat.population.toFixed(0)}万 / 生産性 ${stat.productivity >= 0 ? '+' : ''}${stat.productivity.toFixed(1)}% / 食料 ${(stat.foodAdequacy * 100).toFixed(0)}% / 魅力度 ${(stat.appeal || 50).toFixed(1)} / 交通 ${transport}</span>
          </span>
          <span style="display:flex;gap:4px;flex-wrap:wrap;justify-content:flex-end">${getPrefectureFoodPriorityButtons(prefecture.id)}</span>
        </div>`;
      }).join('');
      return `<div class="research-card">
        <div class="title"><span>${region?.name || regionId}</span><span>${list.length}県</span></div>
        <div class="meta">食料配分・魅力度・生産性をまとめて確認できます。</div>
        ${rows}
      </div>`;
    }).join('');
    return `<div class="section">
      <h3>🍚 県別の食料配分</h3>
      <div class="compact-note" style="margin-bottom:8px">全都道府県を地方別に表示しています。食料が足りず魅力度が低い県から、交通網でつながった魅力的な県へ人口が徐々に移ります。</div>
      ${regionBlocks}
      <div class="research-card" style="margin-top:10px">
        <div class="title"><span>🚚 人口移転政策</span><span>支持率に影響</span></div>
        <div class="meta">国家が強制的に人口を移すこともできますが、支持率と安定度が下がります。交通網があるほど反発は少し軽くなります。</div>
        <div class="save-actions" style="margin-top:10px">
          <select style="flex:1;min-width:120px;padding:8px;background:#0f1530;color:#fff;border:1px solid rgba(255,255,255,.08);border-radius:8px" onchange="setForcedMigrationFrom(this.value)">
            ${prefectures.map(prefecture => `<option value="${prefecture.id}" ${S.forcedMigrationDraft?.from===prefecture.id?'selected':''}>移出: ${prefecture.name}</option>`).join('')}
          </select>
          <select style="flex:1;min-width:120px;padding:8px;background:#0f1530;color:#fff;border:1px solid rgba(255,255,255,.08);border-radius:8px" onchange="setForcedMigrationTo(this.value)">
            ${prefectures.map(prefecture => `<option value="${prefecture.id}" ${S.forcedMigrationDraft?.to===prefecture.id?'selected':''}>移入: ${prefecture.name}</option>`).join('')}
          </select>
        </div>
        <div class="slider-row" style="margin-top:10px">
          <label>移転人数</label>
          <input type="range" min="4" max="60" step="2" value="${S.forcedMigrationDraft?.amount || 18}" oninput="setForcedMigrationAmount(+this.value); this.nextElementSibling.textContent=this.value+'万'">
          <span class="sv" style="width:80px">${S.forcedMigrationDraft?.amount || 18}万</span>
        </div>
        <div class="save-actions"><button class="btn btn-yellow" onclick="executeForcedMigration()">人口移転を実施</button></div>
      </div>
    </div>`;
  }

  function renderPrefectureAllocationSection(prefectureId){
    const prefecture = prefectureMap[prefectureId];
    const stat = getPrefectureStat(prefectureId);
    const region = prefecture ? regions[prefecture.region] : null;
    const activeDisaster = region?.disaster && region.disaster.weeksLeft > 0 ? region.disaster : null;
    if(!prefecture || !stat) return '';
    const localCompanies = getPrefectureCompanies(prefectureId);
    const averagePopulation = getAveragePrefecturePopulation();
    const populationPressure = clamp(stat.population / Math.max(1, averagePopulation), 0.4, 2.5);
    const transportLinks = getPrefectureTransportRoutes(prefectureId, true).length;
    const childShare = Math.max(0, (stat.children || 0) / Math.max(1, stat.population));
    const seniorShare = Math.max(0, (stat.seniors || 0) / Math.max(1, stat.population));
    const economyQuote = getPrefectureEnhancementQuote(prefectureId, 'economy');
    const populationQuote = getPrefectureEnhancementQuote(prefectureId, 'population');
    const brushType = S.prefectureEnhancementBrush?.active ? S.prefectureEnhancementBrush.type : '';
    return `<div class="section">
      <h3>👥 県人口と配分</h3>
      <div class="detail-grid">
        <div class="dg-item"><div class="dg-label">県人口</div><div class="dg-value">${stat.population.toFixed(0)}万</div></div>
        <div class="dg-item"><div class="dg-label">生産性</div><div class="dg-value">${stat.productivity >= 0 ? '+' : ''}${stat.productivity.toFixed(1)}%</div></div>
        <div class="dg-item"><div class="dg-label">食料充足</div><div class="dg-value">${(stat.foodAdequacy * 100).toFixed(0)}%</div></div>
        <div class="dg-item"><div class="dg-label">人口圧力</div><div class="dg-value">${populationPressure.toFixed(2)}x</div></div>
        <div class="dg-item"><div class="dg-label">県内企業</div><div class="dg-value">${localCompanies.length}社</div></div>
        <div class="dg-item"><div class="dg-label">配分モード</div><div class="dg-value">${S.foodAllocationMode === 'auto' ? '自動' : '手動'}</div></div>
        <div class="dg-item"><div class="dg-label">魅力度</div><div class="dg-value">${(stat.appeal || 50).toFixed(1)}</div></div>
        <div class="dg-item"><div class="dg-label">地元の潤い</div><div class="dg-value">${(stat.wealth || 0).toFixed(1)}</div></div>
        <div class="dg-item"><div class="dg-label">交通接続</div><div class="dg-value">${transportLinks}本</div></div>
        <div class="dg-item"><div class="dg-label">災害</div><div class="dg-value">${activeDisaster ? `${activeDisaster.type}` : 'なし'}</div></div>
        <div class="dg-item"><div class="dg-label">子ども</div><div class="dg-value">${(childShare * 100).toFixed(1)}%</div></div>
        <div class="dg-item"><div class="dg-label">高齢者</div><div class="dg-value">${(seniorShare * 100).toFixed(1)}%</div></div>
        <div class="dg-item"><div class="dg-label">出生率</div><div class="dg-value">${Number(stat.birthRate || 0).toFixed(2)}</div></div>
        <div class="dg-item"><div class="dg-label">人口移動</div><div class="dg-value">${(stat.netMigration || 0) >= 0 ? '+' : ''}${(stat.netMigration || 0).toFixed(2)}万</div></div>
      </div>
      <div class="compact-note" style="margin-top:8px">県人口が大きく、生産性・魅力度・交通接続が高いほど、この県の企業は売上と生産効率で有利になります。食料不足や災害は逆方向に効きます。</div>
      ${activeDisaster ? `<div class="compact-note" style="margin-top:6px;color:#ffb3b3">${activeDisaster.type} が継続中のため、県人口・生産性・企業売上に継続ダメージが入っています。残り ${activeDisaster.weeksLeft}週。</div>` : ''}
      <div class="save-actions" style="margin-top:8px">
        ${getPrefectureFoodPriorityButtons(prefectureId)}
      </div>
      <div class="research-card" style="margin-top:10px">
        <div class="title"><span>📈 県強化</span><span>Age of History風</span></div>
        <div class="meta">県を直接強化したり、マップ上でホイール長押しして囲んだ範囲の県をまとめて強化できます。強化には資金と政治資本が必要です。</div>
        <div class="prefecture-invest-grid" style="margin-top:10px">
          <div class="prefecture-invest-card">
            <div class="compact-note">経済強化</div>
            <div style="font-size:18px;font-weight:800;color:#7cf0c0">+${Number(stat.economyBoost || 0).toFixed(1)}</div>
            <div class="compact-note">費用 ${formatMoneyCompact(economyQuote.money)} / 政治 ${economyQuote.pol}</div>
            <div class="save-actions" style="margin-top:8px">
              <button class="btn btn-sm btn-green" onclick="investPrefecture('${prefectureId}','economy')">この県を強化</button>
              <button class="btn btn-sm ${brushType==='economy'?'active':''}" onclick="beginPrefectureEnhancementBrush('economy')">${brushType==='economy'?'範囲選択中':'範囲指定'}</button>
            </div>
          </div>
          <div class="prefecture-invest-card">
            <div class="compact-note">人口成長強化</div>
            <div style="font-size:18px;font-weight:800;color:#ffb27c">+${Number(stat.populationBoost || 0).toFixed(1)}</div>
            <div class="compact-note">費用 ${formatMoneyCompact(populationQuote.money)} / 政治 ${populationQuote.pol}</div>
            <div class="save-actions" style="margin-top:8px">
              <button class="btn btn-sm btn-yellow" onclick="investPrefecture('${prefectureId}','population')">この県を強化</button>
              <button class="btn btn-sm ${brushType==='population'?'active':''}" onclick="beginPrefectureEnhancementBrush('population')">${brushType==='population'?'範囲選択中':'範囲指定'}</button>
            </div>
          </div>
        </div>
        <div class="save-actions" style="margin-top:8px">
          <button class="btn btn-sm" onclick="cancelPrefectureEnhancementBrush()" ${S.prefectureEnhancementBrush?.active ? '' : 'disabled'}>範囲指定を終了</button>
        </div>
      </div>
      <div class="research-card" style="margin-top:10px">
        <div class="title"><span>🏛 県条例</span><span>${stat.ordinanceMode === 'manual' ? '手動' : '自動'}</span></div>
        <div class="meta">少子化、高齢化、企業誘致、住宅政策を県ごとに調整できます。手動にすると、その県だけ直接指定できます。</div>
        <div class="save-actions" style="margin-top:8px">
          <button class="btn btn-sm ${stat.ordinanceMode==='auto'?'btn-blue active':''}" onclick="setPrefectureOrdinanceMode('${prefectureId}','auto')">自動</button>
          <button class="btn btn-sm ${stat.ordinanceMode==='manual'?'active':''}" onclick="setPrefectureOrdinanceMode('${prefectureId}','manual')">手動</button>
        </div>
        ${Object.entries(ORDINANCE_LABELS).map(([key, label]) => `<div class="trade-item" style="align-items:flex-start"><span style="display:flex;flex-direction:column;align-items:flex-start;gap:2px"><strong>${label}</strong><span class="compact-note">${key==='child' ? '出生率と若年流入、子育てコストに影響' : key==='elder' ? '高齢者満足度と社会保障負担に影響' : key==='business' ? '生産性と企業売上に影響' : '魅力度と人口流入に影響'}</span></span><span style="display:flex;gap:4px;flex-wrap:wrap;justify-content:flex-end">${getPrefectureOrdinanceButtons(prefectureId, key)}</span></div>`).join('')}
      </div>
    </div>`;
  }

  function getPrefectureEnhancementQuote(prefectureId, type){
    const stat = getPrefectureStat(prefectureId);
    if(!stat) return {money:0, pol:0};
    if(type === 'population'){
      return {
        money:Math.round(1400 + (stat.populationBoost || 0) * 220 + Math.max(0, (stat.appeal || 50) - 50) * 12),
        pol:Math.round(6 + (stat.populationBoost || 0) * 0.42),
      };
    }
    return {
      money:Math.round(1800 + (stat.economyBoost || 0) * 260 + Math.max(0, stat.productivity || 0) * 35),
      pol:Math.round(8 + (stat.economyBoost || 0) * 0.45),
    };
  }

  function applyPrefectureEnhancement(prefectureId, type, options={}){
    const stat = getPrefectureStat(prefectureId);
    if(!stat || !prefectureMap[prefectureId]) return false;
    const quote = getPrefectureEnhancementQuote(prefectureId, type);
    const moneyCost = Math.max(0, Math.round(quote.money * (options.discount ?? 1)));
    const polCost = Math.max(0, Math.round(quote.pol * (options.discount ?? 1)));
    if(S.money < moneyCost){
      if(!options.silent) addEvent('❌ 資金不足', 'bad');
      return false;
    }
    if(S.polCap < polCost){
      if(!options.silent) addEvent('❌ 政治資本不足', 'bad');
      return false;
    }
    S.money -= moneyCost;
    S.polCap -= polCost;
    if(type === 'population'){
      stat.populationBoost = Number(stat.populationBoost || 0) + 1.4;
      stat.birthRate = Number(stat.birthRate || S.birthRate || 7.4) + 0.08;
      stat.appeal = clamp((stat.appeal || 50) + 0.7, 0, 100);
      stat.population = Math.max(12, stat.population + 0.45 + (stat.populationBoost || 0) * 0.02);
    } else {
      stat.economyBoost = Number(stat.economyBoost || 0) + 1.5;
      stat.productivity = clamp((stat.productivity || 0) + 1.2, -35, 90);
      stat.wealth = clamp((stat.wealth || 0) + 1.0, 0, 100);
    }
    if(!options.silent){
      addEvent(`📈 ${prefectureMap[prefectureId].name} に ${type === 'population' ? '人口成長' : '県経済'} 投資を実施`, 'good');
    }
    return true;
  }

  window.investPrefecture = function(prefectureId, type){
    ensureAllocationState();
    if(!applyPrefectureEnhancement(prefectureId, type)) return;
    updatePrefectureAppealScores();
    applyPrefectureImpactToCompanies();
    renderPanel();
    refreshPrefectureModalIfOpen();
    updateUI();
    drawMap();
  };

  window.beginPrefectureEnhancementBrush = function(type){
    ensureAllocationState();
    S.prefectureEnhancementBrush = {active:true, type:type === 'population' ? 'population' : 'economy'};
    S.prefectureBrushSelection = null;
    if(S.currentMapMode !== 'japan'){
      storeCurrentView();
      S.currentMapMode = 'japan';
      loadViewForMode('japan');
    }
    S.mobileMapHudCollapsed = false;
    updateMapHud();
    drawMap();
    addEvent(`🖍 ${type === 'population' ? '人口成長投資' : '県経済投資'} の範囲選択を開始。マップ上でホイール長押しして囲ってください`, '');
  };

  window.cancelPrefectureEnhancementBrush = function(){
    S.prefectureEnhancementBrush = null;
    S.prefectureBrushSelection = null;
    updateMapHud();
    drawMap();
  };

  window.applyPrefectureEnhancementSelection = function(selection){
    ensureAllocationState();
    const brush = S.prefectureEnhancementBrush;
    if(!brush?.active) return;
    const selected = getPrefecturesInBrushSelection(selection).map(prefecture => prefecture.id);
    if(!selected.length){
      addEvent('❌ 範囲内に県がありません', 'bad');
      S.prefectureBrushSelection = null;
      drawMap();
      return;
    }
    const discount = selected.length >= 3 ? 0.92 : 1;
    const total = selected.reduce((acc, prefectureId) => {
      const quote = getPrefectureEnhancementQuote(prefectureId, brush.type);
      acc.money += Math.round(quote.money * discount);
      acc.pol += Math.round(quote.pol * discount);
      return acc;
    }, {money:0, pol:0});
    if(S.money < total.money){
      addEvent(`❌ 資金不足 (${formatMoneyCompact(total.money)} 必要)`, 'bad');
      S.prefectureBrushSelection = null;
      drawMap();
      return;
    }
    if(S.polCap < total.pol){
      addEvent(`❌ 政治資本不足 (${total.pol} 必要)`, 'bad');
      S.prefectureBrushSelection = null;
      drawMap();
      return;
    }
    selected.forEach(prefectureId => applyPrefectureEnhancement(prefectureId, brush.type, {silent:true, discount}));
    updatePrefectureAppealScores();
    applyPrefectureImpactToCompanies();
    addEvent(`📍 ${selected.length}県へ ${brush.type === 'population' ? '人口成長' : '県経済'} 投資を実施`, 'good');
    S.prefectureBrushSelection = null;
    renderPanel();
    refreshPrefectureModalIfOpen();
    updateUI();
    drawMap();
  };

  function refreshPrefectureModalIfOpen(){
    const modal = document.getElementById('actionModal');
    if(!modal || !modal.classList.contains('show')) return;
    const state = S.prefectureModalState;
    if(!state || !prefectureMap[state.prefectureId]) return;
    const title = document.getElementById('modalTitle')?.textContent || '';
    if(!title.includes(prefectureMap[state.prefectureId].name)) return;
    document.getElementById('modalTitle').textContent = `🗾 ${prefectureMap[state.prefectureId].name}`;
    document.getElementById('modalContent').innerHTML = renderPrefectureModal(state.prefectureId, state.mode || 'prefecture');
  }

  window.setResourceAllocationMode = function(mode){
    ensureAllocationState();
    S.resourceAllocationMode = mode === 'manual' ? 'manual' : 'auto';
    renderPanel();
    refreshPrefectureModalIfOpen();
  };

  window.setFoodAllocationMode = function(mode){
    ensureAllocationState();
    S.foodAllocationMode = mode === 'manual' ? 'manual' : 'auto';
    renderPanel();
    refreshPrefectureModalIfOpen();
  };

  window.setResourceAllocationFocus = function(resourceKey){
    ensureAllocationState();
    if(resources[resourceKey]) S.resourceAllocationFocus = resourceKey;
    renderPanel();
  };

  window.setCompanyResourcePriorityLevel = function(companyName, resourceKey, value){
    setCompanyResourcePriority(companyName, resourceKey, value);
    renderPanel();
    refreshPrefectureModalIfOpen();
  };

  window.setPrefectureFoodPriorityLevel = function(prefectureId, value){
    setPrefectureFoodPriority(prefectureId, value);
    renderPanel();
    refreshPrefectureModalIfOpen();
  };

  window.setPrefectureOrdinanceMode = function(prefectureId, mode){
    const stat = getPrefectureStat(prefectureId);
    if(!stat) return;
    stat.ordinanceMode = mode === 'manual' ? 'manual' : 'auto';
    if(stat.ordinanceMode === 'auto') autoTunePrefectureOrdinances(prefectureId, stat);
    renderPanel();
    refreshPrefectureModalIfOpen();
  };

  window.setPrefectureOrdinanceLevel = function(prefectureId, key, value){
    const stat = getPrefectureStat(prefectureId);
    if(!stat || stat.ordinanceMode !== 'manual' || !ORDINANCE_LABELS[key]) return;
    stat.ordinances[key] = clamp(Number(value), 0, 3);
    renderPanel();
    refreshPrefectureModalIfOpen();
  };

  window.setForcedMigrationFrom = function(prefectureId){
    ensureAllocationState();
    if(prefectureMap[prefectureId]) S.forcedMigrationDraft.from = prefectureId;
    renderPanel();
  };

  window.setForcedMigrationTo = function(prefectureId){
    ensureAllocationState();
    if(prefectureMap[prefectureId]) S.forcedMigrationDraft.to = prefectureId;
    renderPanel();
  };

  window.setForcedMigrationAmount = function(amount){
    ensureAllocationState();
    S.forcedMigrationDraft.amount = clamp(Number(amount), 4, 60);
  };

  window.executeForcedMigration = function(){
    ensureAllocationState();
    const fromId = S.forcedMigrationDraft.from;
    const toId = S.forcedMigrationDraft.to;
    const amount = clamp(Number(S.forcedMigrationDraft.amount || 0), 4, 60);
    if(!prefectureMap[fromId] || !prefectureMap[toId] || fromId === toId){
      addEvent('❌ 人口移転の県指定が不正です', 'bad');
      return;
    }
    const fromStat = getPrefectureStat(fromId);
    const toStat = getPrefectureStat(toId);
    if(!fromStat || !toStat || fromStat.population <= amount + 12){
      addEvent('❌ 移出元の人口余力が足りません', 'bad');
      return;
    }
    const connected = arePrefecturesTransitConnected(fromId, toId);
    movePopulationBetweenPrefectures(fromStat, toStat, amount);
    const approvalLoss = connected ? 0.6 + amount * 0.03 : 1.4 + amount * 0.045;
    S.approval = Math.max(0, S.approval - approvalLoss);
    S.stability = Math.max(0, S.stability - approvalLoss * 0.8);
    S.socialUnrest = clamp(S.socialUnrest + approvalLoss * 0.5, 0, 100);
    addEvent(`🚚 ${prefectureMap[fromId].name} から ${prefectureMap[toId].name} へ ${amount}万 人の移転を実施${connected ? '' : '。交通網が薄く反発が強い'}`, 'bad');
    renderPanel();
    refreshPrefectureModalIfOpen();
    updateUI();
  };

  const rawRenderResources = renderResources;
  renderResources = function(){
    ensureAllocationState();
    try{
      return rawRenderResources() + renderAllocationSummarySection() + renderCompanyAllocationSection() + renderFoodAllocationSection();
    } catch(error){
      reportRuntimeError('renderResources', error);
      try{
        return rawRenderResources();
      } catch(innerError){
        reportRuntimeError('renderResources:base', innerError);
        return `<div class="section"><h3>📦 資源</h3><div class="compact-note">資源画面の描画に失敗しました。いったん別タブへ移動してから戻ってください。</div></div>`;
      }
    }
  };

  const rawRenderPrefectureModal = renderPrefectureModal;
  renderPrefectureModal = function(prefectureId, mode='prefecture'){
    ensureAllocationState();
    return rawRenderPrefectureModal(prefectureId, mode) + renderPrefectureAllocationSection(prefectureId);
  };

  const rawOpenPrefectureModal = openPrefectureModal;
  openPrefectureModal = function(prefectureId, mode='prefecture'){
    ensureAllocationState();
    S.prefectureModalState = {prefectureId, mode};
    return rawOpenPrefectureModal(prefectureId, mode);
  };
  window.openPrefectureModal = openPrefectureModal;

  const rawSwitchPrefectureModalView = window.switchPrefectureModalView;
  window.switchPrefectureModalView = function(prefectureId, mode){
    ensureAllocationState();
    S.prefectureModalState = {prefectureId, mode};
    return rawSwitchPrefectureModalView(prefectureId, mode);
  };

  const rawGameTick = gameTick;
  gameTick = function(options={}){
    ensureAllocationState();
    rawGameTick(options);
    if(!S.started) return;
    ensureAllocationState();
    const moneyBeforeAllocationLayer = S.money;
    progressTransportNetworks();
    const foodResult = allocateFoodForTick();
    const companyResult = allocateCompanyResourcesForTick();
    const transportOps = applyTransportNetworkOperations();
    const foodOpsCost = Math.max(0, foodResult.opsCost || 0);
    const companyOpsCost = Math.max(0, companyResult.opsCost || 0);
    const transportCost = Math.max(0, transportOps?.cost || 0);
    const transportRevenue = Math.max(0, transportOps?.revenue || 0);
    S.money -= (foodOpsCost + companyOpsCost);
    updatePrefectureAppealScores();
    applyPrefectureDemographicsTick();
    applyConnectedPopulationMigration();
    applyPrefectureImpactToCompanies();
    const moneyDeltaFromAllocationLayer = S.money - moneyBeforeAllocationLayer;
    const categorizedAllocationDelta = transportRevenue - transportCost - foodOpsCost - companyOpsCost;
    S.weeklyBreakdown = {
      ...(S.weeklyBreakdown || {}),
      publicRevenue:(S.weeklyBreakdown?.publicRevenue || 0) + transportRevenue,
      operations:(S.weeklyBreakdown?.operations || 0) + foodOpsCost + companyOpsCost + transportCost,
      adjustments:(S.weeklyBreakdown?.adjustments || 0) + (moneyDeltaFromAllocationLayer - categorizedAllocationDelta),
    };
    if(!options.skipUI && typeof updateUI === 'function') updateUI();
  };
  window.gameTick = gameTick;

  ensureAllocationState();
})();

// ============================================================
// CYBER OPERATION MINIGAME LAYER
// ============================================================
(function(){
  function injectCyberGameStyle(){
    if(document.getElementById('cyberGameRuntimeStyle')) return;
    const style = document.createElement('style');
    style.id = 'cyberGameRuntimeStyle';
    style.textContent = `
      .cyber-game-panel{padding:14px;border-radius:16px;background:linear-gradient(180deg,rgba(5,14,26,.96),rgba(7,18,33,.94));border:1px solid rgba(88,181,255,.18);box-shadow:0 14px 34px rgba(0,0,0,.28);min-height:100%;}
      .cyber-game-panel.error-flash{animation:cyberErrorFlash .68s ease;}
      .cyber-game-top{display:flex;justify-content:space-between;align-items:flex-start;gap:10px;flex-wrap:wrap;margin-bottom:10px;}
      .cyber-game-title{font-size:22px;font-weight:800;color:#eef7ff;}
      .cyber-game-meta{font-size:14px;color:#9ed7ff;line-height:1.6;}
      .cyber-game-status{margin:10px 0;padding:10px 12px;border-radius:12px;background:rgba(255,255,255,.05);border:1px solid rgba(255,255,255,.06);font-size:16px;color:#eaf6ff;line-height:1.65;}
      .cyber-error-banner{margin:10px 0;padding:11px 13px;border-radius:12px;background:rgba(255,92,120,.14);border:1px solid rgba(255,126,150,.32);color:#ffd4de;font-size:15px;font-weight:700;box-shadow:0 0 0 1px rgba(255,126,150,.08) inset;}
      .cyber-game-grid{display:grid;gap:8px;}
      .cyber-choice-row{display:flex;gap:8px;flex-wrap:wrap;margin-top:10px;}
      .cyber-choice-row .btn{min-width:56px}
      .cyber-seq-display{display:flex;gap:8px;flex-wrap:wrap;margin:10px 0;}
      .cyber-seq-card{min-width:52px;padding:12px 10px;border-radius:12px;background:rgba(255,255,255,.06);border:1px solid rgba(255,255,255,.09);font-size:20px;text-align:center;color:#f4fbff;}
      .cyber-route-layer{display:flex;gap:10px;justify-content:center;margin:10px 0;}
      .cyber-node-btn{width:72px;height:72px;border-radius:16px;border:1px solid rgba(110,195,255,.24);background:linear-gradient(180deg,rgba(20,43,70,.96),rgba(8,22,37,.94));color:#eaf6ff;font-size:24px;font-weight:800;cursor:pointer;}
      .cyber-node-btn:disabled{opacity:.42;cursor:default;}
      .cyber-packet-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(112px,1fr));gap:8px;margin-top:10px;}
      .cyber-packet-btn{padding:12px;border-radius:12px;border:1px solid rgba(112,195,255,.18);background:rgba(255,255,255,.05);color:#eef8ff;text-align:left;cursor:pointer;}
      .cyber-packet-btn.hit{border-color:rgba(96,255,158,.35);background:rgba(96,255,158,.12);}
      .cyber-packet-btn.miss{border-color:rgba(255,110,134,.3);background:rgba(255,110,134,.12);}
      .cyber-chip-row{display:flex;gap:8px;flex-wrap:wrap;margin-top:10px;}
      .cyber-chip{padding:10px 12px;border-radius:999px;border:1px solid rgba(112,195,255,.18);background:rgba(255,255,255,.05);cursor:pointer;color:#eef8ff;}
      .cyber-chip.active{background:rgba(111,194,255,.18);border-color:rgba(111,194,255,.36);}
      .cyber-mini-badge{display:inline-flex;align-items:center;gap:6px;padding:5px 9px;border-radius:999px;background:rgba(255,255,255,.06);font-size:11px;color:#cce8ff;}
      .cyber-result.good{border-color:rgba(96,255,158,.26);background:linear-gradient(180deg,rgba(10,32,18,.98),rgba(6,20,11,.96));}
      .cyber-result.bad{border-color:rgba(255,110,134,.26);background:linear-gradient(180deg,rgba(34,10,14,.98),rgba(18,5,8,.96));}
      .cyber-workstation{border-radius:18px;overflow:hidden;border:1px solid rgba(98,188,255,.22);box-shadow:0 18px 44px rgba(0,0,0,.34);background:linear-gradient(180deg,rgba(3,10,20,.98),rgba(4,14,24,.96));height:100%;min-height:calc(94vh - 96px);}
      .cyber-osbar{display:flex;align-items:center;justify-content:space-between;gap:10px;padding:10px 14px;background:linear-gradient(90deg,rgba(7,22,38,.98),rgba(10,28,52,.95));border-bottom:1px solid rgba(112,199,255,.18);}
      .cyber-osdots{display:flex;gap:6px;align-items:center;}
      .cyber-osdots span{width:10px;height:10px;border-radius:50%;display:inline-block;}
      .cyber-osdots span:nth-child(1){background:#ff6d7d;}
      .cyber-osdots span:nth-child(2){background:#ffd166;}
      .cyber-osdots span:nth-child(3){background:#4effb0;}
      .cyber-oslabel{font-size:11px;letter-spacing:.14em;color:#d7edff;font-weight:800;text-transform:uppercase;}
      .cyber-desktop{display:grid;grid-template-columns:1.15fr .85fr;gap:0;min-height:calc(94vh - 96px);background:
        linear-gradient(180deg,rgba(2,7,14,.94),rgba(2,8,16,.98)),
        radial-gradient(circle at 20% 20%, rgba(87,196,255,.09), transparent 26%);}
      .cyber-main{padding:18px;border-right:1px solid rgba(255,255,255,.06);position:relative;overflow:auto;min-height:0;}
      .cyber-main::before{content:"";position:absolute;inset:0;pointer-events:none;background:
        linear-gradient(rgba(255,255,255,.03) 1px, transparent 1px),
        linear-gradient(90deg, rgba(255,255,255,.03) 1px, transparent 1px);
        background-size:24px 24px,24px 24px;opacity:.05;}
      .cyber-side{padding:18px;background:linear-gradient(180deg,rgba(4,12,19,.96),rgba(4,10,16,.92));display:flex;flex-direction:column;gap:12px;overflow:auto;min-height:0;}
      .cyber-side-card{padding:12px;border-radius:14px;background:rgba(255,255,255,.04);border:1px solid rgba(255,255,255,.06);}
      .cyber-side-title{font-size:11px;letter-spacing:.1em;text-transform:uppercase;color:#88d5ff;font-weight:800;margin-bottom:8px;}
      .cyber-terminal{margin-top:10px;padding:12px 14px;border-radius:14px;background:linear-gradient(180deg,rgba(2,9,16,.96),rgba(1,6,12,.98));border:1px solid rgba(96,183,255,.16);font-family:Consolas,'Cascadia Code','Courier New',monospace;color:#9bf8c9;}
      .cyber-terminal-line{font-size:16px;line-height:1.75;opacity:.94;}
      .cyber-terminal-line.dim{color:#8bb8cf;opacity:.82;}
      .cyber-terminal-line.warn{color:#ffd18c;}
      .cyber-terminal-line.bad{color:#ff9ab0;}
      .cyber-terminal-line.good{color:#8dffc3;}
      .cyber-briefing{padding:14px;border-radius:16px;background:linear-gradient(180deg,rgba(7,20,34,.98),rgba(4,13,24,.96));border:1px solid rgba(103,190,255,.18);box-shadow:0 10px 24px rgba(0,0,0,.24);}
      .cyber-briefing h4{font-size:24px;color:#f2fbff;margin-bottom:8px;}
      .cyber-briefing p{font-size:16px;line-height:1.8;color:#c7e5ff;}
      .cyber-briefing-step{display:flex;gap:8px;align-items:flex-start;padding:8px 0;border-top:1px solid rgba(255,255,255,.05);}
      .cyber-briefing-step:first-of-type{border-top:none;}
      .cyber-briefing-no{width:28px;height:28px;border-radius:50%;display:flex;align-items:center;justify-content:center;background:rgba(105,190,255,.15);border:1px solid rgba(105,190,255,.28);font-size:14px;font-weight:700;color:#eaf8ff;flex-shrink:0;}
      .cyber-briefing-text{font-size:16px;line-height:1.7;color:#d8ecff;}
      .cyber-hud-grid{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:8px;margin-top:10px;}
      .cyber-hud-tile{padding:10px 12px;border-radius:12px;background:rgba(255,255,255,.04);border:1px solid rgba(255,255,255,.06);}
      .cyber-hud-tile .label{font-size:12px;color:#8ebfda;text-transform:uppercase;letter-spacing:.08em;}
      .cyber-hud-tile .value{font-size:18px;font-weight:800;color:#f4fbff;margin-top:3px;}
      .cyber-start-actions{display:flex;gap:8px;flex-wrap:wrap;margin-top:14px;}
      .cyber-keyboard{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:8px;margin-top:12px;}
      .cyber-keycap{padding:12px 0;border-radius:12px;border:1px solid rgba(111,195,255,.2);background:linear-gradient(180deg,rgba(17,35,58,.96),rgba(10,22,38,.94));color:#edf8ff;font-size:20px;font-weight:800;text-align:center;box-shadow:inset 0 1px 0 rgba(255,255,255,.08);}
      .cyber-scope{height:138px;border-radius:14px;background:linear-gradient(180deg,rgba(3,8,13,.96),rgba(4,10,16,.94));border:1px solid rgba(93,179,255,.16);position:relative;overflow:hidden;}
      .cyber-scope::before{content:"";position:absolute;inset:0;background:repeating-linear-gradient(180deg,rgba(255,255,255,.03),rgba(255,255,255,.03) 1px,transparent 1px,transparent 22px);opacity:.22;}
      .cyber-scope-line{position:absolute;left:-10%;right:-10%;height:2px;top:50%;background:linear-gradient(90deg,transparent,rgba(107,255,179,.16),rgba(107,255,179,.9),rgba(107,255,179,.16),transparent);transform:translateY(-50%);animation:cyberScanLine 2.2s linear infinite;}
      .cyber-scope-pulse{position:absolute;border-radius:999px;border:1px solid rgba(107,255,179,.4);animation:cyberPulse 2.8s ease-out infinite;}
      .cyber-scope-pulse.one{left:18%;top:24%;width:68px;height:68px;}
      .cyber-scope-pulse.two{right:18%;bottom:18%;width:84px;height:84px;animation-delay:1s;}
      .cyber-checksum-values{display:flex;gap:8px;flex-wrap:wrap;margin-top:10px;font-family:Consolas,'Courier New',monospace;}
      .cyber-checksum-box{padding:10px 12px;border-radius:10px;background:rgba(255,255,255,.04);border:1px solid rgba(255,255,255,.06);font-size:16px;color:#eaf7ff;}
      .cyber-action-bar{display:flex;gap:8px;flex-wrap:wrap;margin-top:14px;padding-top:12px;border-top:1px solid rgba(255,255,255,.06);}
      @keyframes cyberErrorFlash{0%{box-shadow:0 0 0 rgba(255,104,126,0);}25%{box-shadow:0 0 0 3px rgba(255,104,126,.22), 0 0 28px rgba(255,104,126,.12);}100%{box-shadow:0 14px 34px rgba(0,0,0,.28);}}
      @keyframes cyberScanLine{0%{transform:translateY(-56px);}100%{transform:translateY(64px);}}
      @keyframes cyberPulse{0%{transform:scale(.55);opacity:.78;}100%{transform:scale(1.35);opacity:0;}}
      @media (max-width: 860px){
        .cyber-desktop{grid-template-columns:1fr;}
        .cyber-main{border-right:none;border-bottom:1px solid rgba(255,255,255,.06);}
      }
    `;
    document.head.appendChild(style);
  }

  function cyberShuffle(list){
    const copy = [...list];
    for(let i = copy.length - 1; i > 0; i--){
      const j = Math.floor(Math.random() * (i + 1));
      [copy[i], copy[j]] = [copy[j], copy[i]];
    }
    return copy;
  }

  function getCyberMiniDifficulty(target){
    const advantage = S.cyberPower - target.cyberPower;
    return {
      advantage,
      maxErrors: advantage >= 22 ? 3 : advantage >= 4 ? 2 : 1,
      sequenceLength: advantage >= 18 ? 4 : advantage >= 3 ? 5 : 6,
      routeLayers: advantage >= 10 ? 4 : 5,
      packetCount: advantage >= 12 ? 6 : 8,
      cipherRounds: advantage >= 12 ? 3 : 4,
      checksumRounds: advantage >= 14 ? 2 : 3,
    };
  }

  function ensureCyberMiniState(){
    if(!S.cyberMiniSeenKinds || typeof S.cyberMiniSeenKinds !== 'object') S.cyberMiniSeenKinds = {};
  }

  function getCyberMiniGuide(kind){
    const guides = {
      sequence:{
        title:'シーケンス・ミラー',
        desc:'画面に一瞬だけ出る侵入コードを覚えて、そのまま同じ順番で入力します。',
        steps:['最初に表示される記号列をそのまま記憶します。','表示が消えたら、同じ順でボタンを押します。','1文字でも違うと検知されます。焦らず順番優先です。'],
      },
      route:{
        title:'ルート・ブリーチ',
        desc:'防壁の層を順番に突破する侵入経路選別です。',
        steps:['各層に安全なノードは1つだけです。','正しいノードを選ぶと次の層へ進みます。','失敗するとその場で検知され、作戦難度が上がります。'],
      },
      packet:{
        title:'パケット・トリアージ',
        desc:'危険パケットだけを抜き出して痕跡を消す選別ゲームです。',
        steps:['危険通信だけを選択します。','正常通信を切ると誤検知でミスになります。','危険パケットをすべて隔離できれば成功です。'],
      },
      cipher:{
        title:'サイファ・クラック',
        desc:'暗号ヒントに対応するキーを選び、段階的に復号します。',
        steps:['表示された暗号に合うキーを1つ選びます。','正解なら次の問題へ進みます。','数問連続で当て切ると侵入成功です。'],
      },
      checksum:{
        title:'チェックサム・バランス',
        desc:'チップの値を足して、目標値ぴったりに合わせます。',
        steps:['表示された数値チップから必要なものを選びます。','合計が目標値ぴったりで次の問題へ進みます。','過不足があると検知されるので、送信前に合計を確認します。'],
      },
    };
    return guides[kind] || {title:'侵入作戦', desc:'侵入手順を進めてください。', steps:[]};
  }

  function buildSequenceGame(targetIdx, diff){
    const symbols = ['◈','⬢','✦','⬤','▲','■'];
    return {
      kind:'sequence',
      title:'シーケンス・ミラー',
      summary:'表示された侵入コードを記憶して、同じ順に打ち込みます。',
      targetIdx,
      errors:0,
      maxErrors:diff.maxErrors,
      sequence:Array.from({length:diff.sequenceLength}, () => symbols[Math.floor(Math.random() * symbols.length)]),
      entered:[],
      symbolSet:symbols,
      revealed:true,
      hint:'コードを数秒だけ表示します。隠れたら同じ順番で押してください。',
    };
  }

  function buildRouteGame(targetIdx, diff){
    return {
      kind:'route',
      title:'ルート・ブリーチ',
      summary:'層ごとに正しい侵入ノードを選んで突破します。',
      targetIdx,
      errors:0,
      maxErrors:diff.maxErrors,
      layers:Array.from({length:diff.routeLayers}, () => Math.floor(Math.random() * 3)),
      currentLayer:0,
      labels:['A','B','C'],
      hint:'各層に1つだけ安全な経路があります。',
    };
  }

  function buildPacketGame(targetIdx, diff){
    const count = diff.packetCount;
    const maliciousCount = Math.max(3, Math.round(count * 0.5));
    const labels = cyberShuffle(['VX-1','PX-9','GH-3','KT-2','RM-7','JS-4','UL-8','DN-5']).slice(0, count);
    const maliciousIndexes = new Set(cyberShuffle(Array.from({length:count}, (_, index) => index)).slice(0, maliciousCount));
    return {
      kind:'packet',
      title:'パケット・トリアージ',
      summary:'危険パケットだけを摘出して、通信ログをきれいにします。',
      targetIdx,
      errors:0,
      maxErrors:diff.maxErrors,
      packets:labels.map((label, index) => ({id:index, label, malicious:maliciousIndexes.has(index), state:'idle'})),
      hint:'危険な通信だけを選んで隔離してください。',
    };
  }

  function buildCipherGame(targetIdx, diff){
    const pools = [
      {prompt:'A7-X', correct:'Lotus', options:['Lotus','Amber','Crown','Helix']},
      {prompt:'K2-V', correct:'Aegis', options:['Orbit','Aegis','Prism','Flare']},
      {prompt:'Q9-D', correct:'Prism', options:['Signal','Prism','Comet','Quartz']},
      {prompt:'R1-N', correct:'Helix', options:['Delta','Vector','Helix','Pulse']},
      {prompt:'T4-H', correct:'Crown', options:['Crown','Vapor','Shell','Crane']},
    ];
    return {
      kind:'cipher',
      title:'サイファ・クラック',
      summary:'暗号ヒントに合う復号キーを見つけます。',
      targetIdx,
      errors:0,
      maxErrors:diff.maxErrors,
      rounds:cyberShuffle(pools).slice(0, diff.cipherRounds),
      currentRound:0,
      hint:'正しいキーを選ぶたびに次の暗号へ進みます。',
    };
  }

  function buildChecksumGame(targetIdx, diff){
    const rounds = [];
    for(let i = 0; i < diff.checksumRounds; i++){
      const chips = cyberShuffle([2,3,4,5,6,7,8]).slice(0, 5);
      const combo = cyberShuffle(chips).slice(0, 2 + (i % 2));
      rounds.push({
        chips,
        target:combo.reduce((sum, value) => sum + value, 0),
      });
    }
    return {
      kind:'checksum',
      title:'チェックサム・バランス',
      summary:'必要な値ちょうどになるようにチップを選びます。',
      targetIdx,
      errors:0,
      maxErrors:diff.maxErrors,
      rounds,
      currentRound:0,
      selected:[],
      hint:'過不足なく目標値に合わせると次の段階へ進みます。',
    };
  }

  function createCyberMiniGame(targetIdx){
    ensureCyberMiniState();
    const target = tradePartners[targetIdx];
    const diff = getCyberMiniDifficulty(target);
    const factory = pick([
      () => buildSequenceGame(targetIdx, diff),
      () => buildRouteGame(targetIdx, diff),
      () => buildPacketGame(targetIdx, diff),
      () => buildCipherGame(targetIdx, diff),
      () => buildChecksumGame(targetIdx, diff),
    ]);
    const game = factory();
    game.id = `cyber-${Date.now()}-${Math.random().toString(36).slice(2,6)}`;
    game.targetName = target.name;
    game.advantage = diff.advantage;
    game.result = null;
    game.perfect = false;
    game.showBriefing = !S.cyberMiniSeenKinds[game.kind];
    return game;
  }

  function clearCyberMiniTimer(){
    if(window.__cyberMiniTimer){
      clearTimeout(window.__cyberMiniTimer);
      window.__cyberMiniTimer = 0;
    }
    if(window.__cyberMiniFlashTimer){
      clearTimeout(window.__cyberMiniFlashTimer);
      window.__cyberMiniFlashTimer = 0;
    }
  }

  function scheduleSequenceHide(){
    clearCyberMiniTimer();
    const game = S.cyberMiniGame;
    if(!game || game.kind !== 'sequence' || !game.revealed || game.showBriefing) return;
    const gameId = game.id;
    window.__cyberMiniTimer = setTimeout(() => {
      if(!S.cyberMiniGame || S.cyberMiniGame.id !== gameId || S.cyberMiniGame.kind !== 'sequence') return;
      S.cyberMiniGame.revealed = false;
      renderCyberMiniGameModal();
    }, 5600 + game.sequence.length * 920);
  }

  function failCyberMiniGame(reason){
    const game = S.cyberMiniGame;
    if(!game) return;
    game.result = {success:false, text:reason || '侵入が露見しました。逆探知を受けています。'};
    finalizeCyberMiniGame(false);
  }

  function markCyberMiniError(reason){
    const game = S.cyberMiniGame;
    if(!game || game.result) return;
    game.errors += 1;
    game.status = reason;
    game.lastErrorAt = Date.now();
    if(game.errors > game.maxErrors){
      failCyberMiniGame(reason || '操作ミスが重なりました。');
      return;
    }
    if(window.__cyberMiniFlashTimer) clearTimeout(window.__cyberMiniFlashTimer);
    window.__cyberMiniFlashTimer = setTimeout(() => {
      if(!S.cyberMiniGame || S.cyberMiniGame.id !== game.id) return;
      S.cyberMiniGame.lastErrorAt = 0;
      renderCyberMiniGameModal();
    }, 760);
    renderCyberMiniGameModal();
  }

  function finalizeCyberMiniGame(success){
    const game = S.cyberMiniGame;
    if(!game) return;
    const target = tradePartners[game.targetIdx];
    if(!target) return;
    clearCyberMiniTimer();
    const flawless = game.errors === 0;
    game.perfect = flawless;
    if(success){
      const bonus = flawless ? 1.24 : 1;
      const steal = Math.round((280 + target.economy * rand(7, 16) + S.techLevel * 3) * bonus);
      const marketHit = rand(0.96, 0.99);
      S.money += steal;
      S.cyberSuccesses = (S.cyberSuccesses || 0) + 1;
      target.relation = Math.max(0, target.relation - (flawless ? 1 : 2));
      target.stability = Math.max(0, target.stability - (flawless ? 3 : 5));
      target.economy = Math.max(10, target.economy - (flawless ? 2 : 4));
      target.gdp = Math.max(120, target.gdp * (flawless ? 0.992 : 0.986));
      target.population = Math.max(5, target.population - (flawless ? 0.1 : 0.3));
      if(Math.random() < 0.45) target.cyberPower = Math.max(0, target.cyberPower - 1);
      companies.filter(company => ['tech','defense','telecom'].includes(company.sector)).forEach(company => {
        company.price = Math.max(100, Math.round(company.price * rand(1.008, 1.028)));
        company.revenue *= rand(1.002, 1.012);
      });
      if(Array.isArray(S.foreignCompanies)){
        S.foreignCompanies.filter(company => company.countryId === target.id).forEach(company => {
          company.price = Math.max(100, Math.round(company.price * marketHit));
          company.revenue *= rand(0.986, 0.996);
        });
      }
      game.result = {success:true, text:`侵入成功。${target.name} から ${moneyCompact(steal)} を獲得し、相手国の経済と安定度に打撃を与えました。`};
      addEvent(`🕶 ${target.name}へのサイバー侵入成功。${moneyCompact(steal)}を獲得`, 'good');
    } else {
      const loss = Math.round(180 + target.economy * rand(5, 11));
      S.money = Math.max(0, S.money - loss);
      S.approval = Math.max(0, S.approval - 1.2);
      target.relation = Math.max(0, target.relation - 5);
      target.stability = Math.min(100, target.stability + 2);
      target.cyberPower = Math.min(160, target.cyberPower + 2);
      game.result = {success:false, text:`作戦失敗。逆探知で ${moneyCompact(loss)} の損失と支持率低下が発生しました。`};
      addEvent(`🛑 ${target.name}へのサイバー侵入失敗。逆探知を受けた`, 'bad');
    }
    renderCyberMiniGameModal();
    updateUI();
    renderPanel();
  }

  function renderCyberMissionShell(game, innerHtml){
    const sideLog = [
      `TARGET :: ${game.targetName}`,
      `THREAD :: ${game.title}`,
      `SIGNAL :: ${game.advantage >= 12 ? 'STABLE' : game.advantage >= 0 ? 'FRINGE' : 'CONTESTED'}`,
      `COST :: ${moneyCompact(game.moneyCost || 0)} / POL ${game.polCost || 0}`,
      game.status || '侵入準備完了',
    ];
    return `<div class="cyber-workstation">
      <div class="cyber-osbar">
        <div class="cyber-osdots"><span></span><span></span><span></span></div>
        <div class="cyber-oslabel">Ops Console / GhostLink</div>
        <div class="cyber-mini-badge">Live Session</div>
      </div>
      <div class="cyber-desktop">
        <div class="cyber-main">${innerHtml}</div>
        <div class="cyber-side">
          <div class="cyber-side-card">
            <div class="cyber-side-title">Mission Feed</div>
            <div class="cyber-terminal">
              ${sideLog.map((line, index) => `<div class="cyber-terminal-line ${index === sideLog.length - 1 ? 'good' : 'dim'}">&gt; ${line}</div>`).join('')}
            </div>
          </div>
          <div class="cyber-side-card">
            <div class="cyber-side-title">Network Scope</div>
            <div class="cyber-scope">
              <div class="cyber-scope-line"></div>
              <div class="cyber-scope-pulse one"></div>
              <div class="cyber-scope-pulse two"></div>
            </div>
          </div>
          <div class="cyber-side-card">
            <div class="cyber-side-title">Operational Notes</div>
            <div class="compact-note">成功すると資金奪取と対象国ダメージ。完璧に成功すると関係悪化は軽めです。失敗すると逆探知で損失と支持率低下が入ります。今回の投入資金は ${moneyCompact(game.moneyCost || 0)}、政治資本は ${game.polCost || 0} です。</div>
          </div>
        </div>
      </div>
    </div>`;
  }

  function renderCyberBriefing(game){
    const guide = getCyberMiniGuide(game.kind);
    return renderCyberMissionShell(game, `<div class="cyber-briefing">
      <h4>${guide.title}</h4>
      <p>${guide.desc}</p>
      ${guide.steps.map((step, index) => `<div class="cyber-briefing-step"><div class="cyber-briefing-no">${index + 1}</div><div class="cyber-briefing-text">${step}</div></div>`).join('')}
      <div class="cyber-hud-grid">
        <div class="cyber-hud-tile"><div class="label">Target</div><div class="value">${game.targetName}</div></div>
        <div class="cyber-hud-tile"><div class="label">Mistakes</div><div class="value">${game.maxErrors} 回まで</div></div>
        <div class="cyber-hud-tile"><div class="label">Advantage</div><div class="value">${Math.round(game.advantage)}</div></div>
        <div class="cyber-hud-tile"><div class="label">Mode</div><div class="value">${game.kind.toUpperCase()}</div></div>
      </div>
      <div class="cyber-start-actions">
        <button class="btn btn-green" onclick="cyberMiniDismissBriefing()">侵入開始</button>
        <button class="btn" onclick="closeCyberMiniGame()">中止</button>
      </div>
    </div>`);
  }

  function renderCyberMiniGameBody(game){
    const errorFlash = game.lastErrorAt && Date.now() - game.lastErrorAt < 900;
    const errorClass = errorFlash ? 'error-flash' : '';
    const errorBanner = errorFlash ? `<div class="cyber-error-banner">${game.status || '検知されました。操作を見直してください。'}</div>` : '';
    if(game.result){
      return renderCyberMissionShell(game, `<div class="cyber-game-panel cyber-result ${game.result.success ? 'good' : 'bad'}">
        <div class="cyber-game-title">${game.result.success ? '侵入成功' : '侵入失敗'}</div>
        <div class="cyber-game-status">${game.result.text}</div>
        <div class="save-actions" style="margin-top:10px">
          <button class="btn btn-sm btn-blue" onclick="closeCyberMiniGame()">閉じる</button>
        </div>
      </div>`);
    }
    if(game.showBriefing) return renderCyberBriefing(game);
    const badges = `<span class="cyber-mini-badge">対象 ${game.targetName}</span><span class="cyber-mini-badge">ミス ${game.errors}/${game.maxErrors}</span><span class="cyber-mini-badge">有利差 ${Math.round(game.advantage)}</span>`;
    if(game.kind === 'sequence'){
      return renderCyberMissionShell(game, `<div class="cyber-game-panel ${errorClass}">
        <div class="cyber-game-top"><div><div class="cyber-game-title">${game.title}</div><div class="cyber-game-meta">${game.summary}</div></div><div class="cyber-choice-row">${badges}</div></div>
        <div class="cyber-game-status">${game.revealed ? 'コード表示中。覚えたら入力に移ります。' : game.status || game.hint}</div>
        ${errorBanner}
        <div class="cyber-seq-display">${game.sequence.map((symbol, index) => `<div class="cyber-seq-card">${game.revealed || index < game.entered.length ? symbol : '•'}</div>`).join('')}</div>
        <div class="cyber-keyboard">${game.symbolSet.map(symbol => `<button class="cyber-keycap" ${game.revealed ? 'disabled' : ''} onclick="cyberMiniPressSymbol('${symbol}')">${symbol}</button>`).join('')}</div>
      </div>`);
    }
    if(game.kind === 'route'){
      return renderCyberMissionShell(game, `<div class="cyber-game-panel ${errorClass}">
        <div class="cyber-game-top"><div><div class="cyber-game-title">${game.title}</div><div class="cyber-game-meta">${game.summary}</div></div><div class="cyber-choice-row">${badges}</div></div>
        <div class="cyber-game-status">${game.status || `${game.currentLayer + 1}層目のノードを選択してください。`}</div>
        ${errorBanner}
        ${game.layers.map((safe, layerIndex) => `<div class="cyber-route-layer">${game.labels.map((label, optionIndex) => {
          const done = layerIndex < game.currentLayer;
          const active = layerIndex === game.currentLayer;
          return `<button class="cyber-node-btn" ${!active ? 'disabled' : ''} onclick="cyberMiniChooseRoute(${optionIndex})">${done && optionIndex === safe ? '✓' : label}</button>`;
        }).join('')}</div>`).join('')}
      </div>`);
    }
    if(game.kind === 'packet'){
      return renderCyberMissionShell(game, `<div class="cyber-game-panel ${errorClass}">
        <div class="cyber-game-top"><div><div class="cyber-game-title">${game.title}</div><div class="cyber-game-meta">${game.summary}</div></div><div class="cyber-choice-row">${badges}</div></div>
        <div class="cyber-game-status">${game.status || game.hint}</div>
        ${errorBanner}
        <div class="cyber-packet-grid">${game.packets.map(packet => `<button class="cyber-packet-btn ${packet.state === 'hit' ? 'hit' : packet.state === 'miss' ? 'miss' : ''}" ${packet.state !== 'idle' ? 'disabled' : ''} onclick="cyberMiniScanPacket(${packet.id})"><strong>${packet.label}</strong><br><span class="compact-note">${packet.state === 'hit' ? '隔離済み' : packet.state === 'miss' ? '誤検知' : '未判定'}</span></button>`).join('')}</div>
      </div>`);
    }
    if(game.kind === 'cipher'){
      const round = game.rounds[game.currentRound];
      return renderCyberMissionShell(game, `<div class="cyber-game-panel ${errorClass}">
        <div class="cyber-game-top"><div><div class="cyber-game-title">${game.title}</div><div class="cyber-game-meta">${game.summary}</div></div><div class="cyber-choice-row">${badges}<span class="cyber-mini-badge">${game.currentRound + 1}/${game.rounds.length}</span></div></div>
        <div class="cyber-game-status">${game.status || `暗号 ${round.prompt} に対応するキーを選んでください。`}</div>
        ${errorBanner}
        <div class="cyber-choice-row">${round.options.map(option => `<button class="btn btn-sm" onclick="cyberMiniChooseCipher('${option}')">${option}</button>`).join('')}</div>
      </div>`);
    }
    if(game.kind === 'checksum'){
      const round = game.rounds[game.currentRound];
      const total = game.selected.reduce((sum, index) => sum + round.chips[index], 0);
      return renderCyberMissionShell(game, `<div class="cyber-game-panel ${errorClass}">
        <div class="cyber-game-top"><div><div class="cyber-game-title">${game.title}</div><div class="cyber-game-meta">${game.summary}</div></div><div class="cyber-choice-row">${badges}<span class="cyber-mini-badge">${game.currentRound + 1}/${game.rounds.length}</span></div></div>
        <div class="cyber-game-status">${game.status || `目標値 ${round.target} にぴったり合わせてください。現在 ${total}`}</div>
        ${errorBanner}
        <div class="cyber-checksum-values">
          <div class="cyber-checksum-box">目標値 ${round.target}</div>
          <div class="cyber-checksum-box">現在値 ${total}</div>
          <div class="cyber-checksum-box">段階 ${game.currentRound + 1}/${game.rounds.length}</div>
        </div>
        <div class="cyber-chip-row">${round.chips.map((value, index) => {
          const active = game.selected.includes(index);
          return `<button class="cyber-chip ${active ? 'active' : ''}" onclick="cyberMiniToggleChecksum(${index})">${value}</button>`;
        }).join('')}</div>
        <div class="cyber-action-bar">
          <button class="btn btn-sm btn-green" onclick="cyberMiniSubmitChecksum()">送信</button>
          <button class="btn btn-sm" onclick="cyberMiniClearChecksum()">クリア</button>
        </div>
      </div>`);
    }
    return '';
  }

  function renderCyberMiniGameModal(){
    const game = S.cyberMiniGame;
    if(!game) return;
  injectCyberGameStyle();
  const modal = document.getElementById('actionModal');
  const modalCard = modal.querySelector('.modal');
  if(modalCard){
    modalCard.classList.remove('modal-world-country');
    modalCard.classList.add('modal-cyber');
  }
  modal.dataset.action = 'cyber-mini';
  document.getElementById('modalTitle').textContent = `🕶 ${game.targetName} 侵入作戦`;
    document.getElementById('modalContent').innerHTML = renderCyberMiniGameBody(game);
    modal.classList.add('show');
  }

  window.closeCyberMiniGame = function(){
    clearCyberMiniTimer();
    S.cyberMiniGame = null;
    closeModal();
  };

  window.cyberMiniDismissBriefing = function(){
    ensureCyberMiniState();
    const game = S.cyberMiniGame;
    if(!game) return;
    S.cyberMiniSeenKinds[game.kind] = true;
    game.showBriefing = false;
    game.status = '侵入手順を開始します。';
    renderCyberMiniGameModal();
    if(game.kind === 'sequence') scheduleSequenceHide();
  };

  window.cyberMiniPressSymbol = function(symbol){
    const game = S.cyberMiniGame;
    if(!game || game.kind !== 'sequence' || game.revealed) return;
    const expected = game.sequence[game.entered.length];
    if(symbol !== expected){
      markCyberMiniError(`誤ったコードを入力しました。正解は ${expected} ではありません。`);
      return;
    }
    game.entered.push(symbol);
    game.status = `${game.entered.length}/${game.sequence.length} 正解`;
    if(game.entered.length >= game.sequence.length){
      finalizeCyberMiniGame(true);
      return;
    }
    renderCyberMiniGameModal();
  };

  window.cyberMiniChooseRoute = function(optionIndex){
    const game = S.cyberMiniGame;
    if(!game || game.kind !== 'route' || game.result) return;
    const safe = game.layers[game.currentLayer];
    if(optionIndex !== safe){
      markCyberMiniError(`第${game.currentLayer + 1}層で防壁に接触しました。`);
      return;
    }
    game.currentLayer += 1;
    game.status = `${game.currentLayer}/${game.layers.length} 層突破`;
    if(game.currentLayer >= game.layers.length){
      finalizeCyberMiniGame(true);
      return;
    }
    renderCyberMiniGameModal();
  };

  window.cyberMiniScanPacket = function(packetId){
    const game = S.cyberMiniGame;
    if(!game || game.kind !== 'packet' || game.result) return;
    const packet = game.packets.find(entry => entry.id === packetId);
    if(!packet || packet.state !== 'idle') return;
    if(packet.malicious){
      packet.state = 'hit';
      game.status = `危険通信 ${packet.label} を隔離しました。`;
      const allHit = game.packets.filter(entry => entry.malicious).every(entry => entry.state === 'hit');
      if(allHit){
        finalizeCyberMiniGame(true);
        return;
      }
    } else {
      packet.state = 'miss';
      markCyberMiniError(`正常通信 ${packet.label} を誤って遮断しました。`);
      return;
    }
    renderCyberMiniGameModal();
  };

  window.cyberMiniChooseCipher = function(option){
    const game = S.cyberMiniGame;
    if(!game || game.kind !== 'cipher' || game.result) return;
    const round = game.rounds[game.currentRound];
    if(option !== round.correct){
      markCyberMiniError(`キー ${option} は不正確でした。`);
      return;
    }
    game.currentRound += 1;
    game.status = `暗号解読 ${game.currentRound}/${game.rounds.length}`;
    if(game.currentRound >= game.rounds.length){
      finalizeCyberMiniGame(true);
      return;
    }
    renderCyberMiniGameModal();
  };

  window.cyberMiniToggleChecksum = function(index){
    const game = S.cyberMiniGame;
    if(!game || game.kind !== 'checksum' || game.result) return;
    if(game.selected.includes(index)) game.selected = game.selected.filter(entry => entry !== index);
    else game.selected.push(index);
    renderCyberMiniGameModal();
  };

  window.cyberMiniClearChecksum = function(){
    const game = S.cyberMiniGame;
    if(!game || game.kind !== 'checksum') return;
    game.selected = [];
    renderCyberMiniGameModal();
  };

  window.cyberMiniSubmitChecksum = function(){
    const game = S.cyberMiniGame;
    if(!game || game.kind !== 'checksum' || game.result) return;
    const round = game.rounds[game.currentRound];
    if(!round){
      finalizeCyberMiniGame(true);
      return;
    }
    const total = game.selected.reduce((sum, index) => sum + round.chips[index], 0);
    if(total !== round.target){
      game.selected = [];
      markCyberMiniError(`値が ${total} になりました。目標 ${round.target} に届いていません。`);
      return;
    }
    game.currentRound += 1;
    game.selected = [];
    game.status = game.currentRound >= game.rounds.length ? '全段階の整合を確認しました。' : `第${game.currentRound + 1}問へ移行。新しい目標値を確認してください。`;
    if(game.currentRound >= game.rounds.length){
      finalizeCyberMiniGame(true);
      return;
    }
    renderCyberMiniGameModal();
  };

  const rawRenderCyber = renderCyber;
  renderCyber = function(){
    let html = `<div class="section"><h3>💻 サイバー戦力 (${Math.round(S.cyberPower)})</h3>
      <p style="font-size:8px;color:#666">投資でサイバー力を上げ、侵入作戦はミニゲームで解決します。成功すると資金を奪い、敵国の安定と経済を傷つけます。</p>
      <div style="padding:6px;background:rgba(0,0,0,.2);border-radius:4px;margin:4px 0">
        <div style="font-size:10px;font-weight:bold">🔒 サイバー防衛強化</div>
        <button class="btn btn-sm btn-purple" onclick="investCyber(12000)">投資12,000億 (+4〜6)</button>
        <button class="btn btn-sm btn-purple" onclick="investCyber(28000)">投資28,000億 (+9〜13)</button>
        <button class="btn btn-sm btn-purple" onclick="investCyber(52000)">大規模投資52,000億 (+17〜24)</button>
      </div>
      <div class="headline-card" style="margin-top:8px">
        <div class="meta">侵入ゲーム</div>
        <div class="body">シーケンス記憶 / ルート突破 / パケット隔離 / 暗号解読 / チェックサム調整 の5種類からランダムで出ます。</div>
      </div>
      <div style="margin-top:8px">
        <div style="font-size:10px;font-weight:bold;margin-bottom:4px">各国サイバー力比較</div>`;
    tradePartners.forEach(tp => {
      const advantage = S.cyberPower > tp.cyberPower;
      html += `<div class="trade-item"><span>${tp.name}</span><span style="color:${advantage ? '#4ecca3' : '#e94560'}">${Math.round(tp.cyberPower)} ${advantage ? '◀ 優勢' : '▶ 劣勢'}</span></div>`;
    });
    html += `</div><div class="section" style="margin-top:8px"><h3>🕶 侵入作戦</h3>`;
    tradePartners.forEach((tp, i) => {
      const edge = S.cyberPower - tp.cyberPower;
      const comfort = edge >= 15 ? '易' : edge >= 0 ? '並' : '難';
      const moneyCost = Math.round(2200 + tp.economy * 20 + tp.cyberPower * 8);
      const polCost = Math.round(24 + tp.cyberPower * 0.08);
      html += `<div class="trade-item"><span>${tp.name}</span><span><button class="btn btn-sm" onclick="launchCyberAttack(${i})">作戦開始 ${comfort} / ${moneyCompact(moneyCost)} / 🏛${polCost}</button></span></div>`;
    });
    if(S.cyberMiniGame){
      html += `<div class="compact-note" style="margin-top:8px">進行中: ${S.cyberMiniGame.targetName} / ${S.cyberMiniGame.title}</div>`;
    }
    html += '</div></div>';
    return html;
  };

  window.launchCyberAttack = function(targetIdx){
    const target = tradePartners[targetIdx];
    if(!target) return;
    const moneyCost = Math.round(2200 + target.economy * 20 + target.cyberPower * 8);
    const polCost = Math.round(24 + target.cyberPower * 0.08);
    if(S.money < moneyCost){ addEvent('❌ 資金不足', 'bad'); return; }
    if(S.polCap < polCost){ addEvent('❌ 政治資本不足', 'bad'); return; }
    S.money -= moneyCost;
    S.polCap -= polCost;
    runWithViewLoader('サイバー侵入環境を準備中...', () => {
      S.cyberMiniGame = createCyberMiniGame(targetIdx);
      if(S.cyberMiniGame){
        S.cyberMiniGame.moneyCost = moneyCost;
        S.cyberMiniGame.polCost = polCost;
      }
      renderCyberMiniGameModal();
      if(S.cyberMiniGame?.kind === 'sequence' && !S.cyberMiniGame.showBriefing) scheduleSequenceHide();
    });
    addEvent(`🕶 ${target.name} へ侵入作戦を開始。費用 ${moneyCompact(moneyCost)} / 政治資本 ${polCost}`, '');
    updateUI();
    renderPanel();
  };
})();

S.loopStarted = true;
requestAnimationFrame(gameLoop);
</script>
</body>
</html>

