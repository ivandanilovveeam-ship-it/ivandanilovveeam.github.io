
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Citadel Estimator</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
<style>
*{box-sizing:border-box;margin:0;padding:0}
body{font-family:system-ui,-apple-system,sans-serif;font-size:12px;background:#F1F5F9}
input[type=number]::-webkit-outer-spin-button,input[type=number]::-webkit-inner-spin-button{-webkit-appearance:none}
input[type=number]{-moz-appearance:textfield}
::-webkit-scrollbar{width:4px;height:4px}
::-webkit-scrollbar-thumb{background:rgba(0,0,0,0.15);border-radius:2px}
select,button,input{font-family:inherit}
a{text-decoration:none}
</style>
</head>
<body>
<div id="root" style="height:100vh;overflow:hidden"></div>
<script>
const{useState,useMemo,useRef,createElement:el}=React;

// ── Catalog JSON format (for Import Catalog Items button):
// [{"cat":"Camera","desc":"Product name","part":"PART-NO","sup":"Supplier","cost":299,"url":"https://..."}]
//
// ── Project JSON format (Save/Load Project):
// {"proj":{...},"labour":[...],"equip":[...],"matl":[...],"addl":[...],"cfg":{...},"savedAt":"..."}

const UBI={
  "UVC-G6-Dome-W":"https://ca.store.ui.com/ca/en/products/uvc-g6-dome",
  "UVC-G6-Mini-Dome-W":"https://ca.store.ui.com/ca/en/products/uvc-g6-mini-dome",
  "UBI-UDM-PRO":"https://ca.store.ui.com/ca/en/products/udm-pro",
  "UBI-USW-48-POE":"https://ca.store.ui.com/ca/en/products/usw-48-poe",
  "UBI-U7-PRO-US":"https://ca.store.ui.com/ca/en/products/u7-pro",
  "UNVR-G2":"https://ca.store.ui.com/ca/en/products/unvr-g2",
  "EAH-8":"https://ca.store.ui.com/ca/en/products/eah-8",
  "UA-G3-PRO-W":"https://ca.store.ui.com/ca/en/products/ua-g3-pro",
  "UA-G3-Flex-W":"https://ca.store.ui.com/ca/en/products/ua-g3-flex",
  "UA-G3-W":"https://ca.store.ui.com/ca/en/products/ua-g3",
  "UA-LOCK-MAGNETIC-270KG":"https://ca.store.ui.com/ca/en/products/ua-lock-magnetic",
  "USP-PDU-Pro":"https://ca.store.ui.com/ca/en/products/usp-pdu-pro",
  "UBI-USW-Flex-2.5G-8-PoE":"https://ca.store.ui.com/ca/en/products/usw-flex-2-5g-8-poe",
  "UBI-USW-Pro-Max-24-PoE":"https://ca.store.ui.com/ca/en/products/usw-pro-max-24-poe",
  "UBI-USW-Pro-Max-16-PoE":"https://ca.store.ui.com/ca/en/products/usw-pro-max-16-poe",
  "UBI-USW-Pro-Max-48-PoE":"https://ca.store.ui.com/ca/en/products/usw-pro-max-48-poe",
};

const DEF_CAT=[
  ["Camera","UniFi G6 Dome — 4K 8MP all-weather, IK10, vandal-proof, AI","UVC-G6-Dome-W","XLR Security",346],
  ["Camera","UniFi G6 Mini Dome — 4K 8MP indoor, AI, two-way audio, IR","UVC-G6-Mini-Dome-W","Ubiquiti CA",429],
  ["Camera","UniFi G6 Turret 4K PoE, 8MP, weatherproof","UBI-UVC-G6-Turret-W","XLR Security",305],
  ["Camera","UniFi G6 Pro Bullet 4K PoE+, 2.36x optical zoom — White","UBI-UVC-G6-Pro-Bullet-W","XLR Security",738],
  ["Camera","UniFi G6 Pro Bullet 4K PoE+, 2.36x optical zoom — Black","UBI-UVC-G6-Pro-Bullet-B","XLR Security",738],
  ["Camera","UniFi G6 Bullet 4K PoE, 8MP all-weather, AI, long-range IR — White","UBI-UVC-G6-Bullet-W","XLR Security",317],
  ["Camera","UniFi G6 Entry — Door entry, Protect + Access","UBI-UVC-G6-Entry","XLR Security",395],
  ["Camera","UniFi G5 Ultra Turret, 2K HD PoE, Long-range Night Vision","UBI-UVC-G5-TURRET-ULTRA","XLR Security",187],
  ["Camera","UniFi UVC AI Pro, 4K PoE, 3x Optical Zoom","UBI-UVC-AI-Pro","XLR Security",754],
  ["Camera","UniFi UVC AI Turret (Black), 4K PoE+, Enhanced AI","UBI-UVC-AI-Turret-B","XLR Security",599],
  ["Camera","UniFi UVC AI PTZ (Black), 4K PoE++, 22x Zoom","UBI-UVC-AI-PTZ-B","XLR Security",2119],
  ["Camera","UniFi G4 Doorbell Pro PoE Kit","UBI-UVC-G4-Doorbell-Pro-PoE-Kit","XLR Security",570],
  ["Camera","Uniview 8MP Turret, Fixed 2.8mm, IP67, Mic","IPC3618SR-ADF28KM-H","XLR Security",182],
  ["Camera","Uniview 8MP Tri-Guard 3.0 Varifocal 2.8-12mm, IP67","IPC3638SE-ADZKMC-WP-I1","XLR Security",466],
  ["Camera","Uniview 8MP LightHunter Dome, Varifocal 2.8-12mm, IK10","IPC3238SB-ADZK-I0","XLR Security",411],
  ["Camera","Uniview 12MP 360° Fisheye, IR 15m, Mic+Speaker","IPC86CEB-AF18KC-I0","XLR Security",648],
  ["Camera","Uniview IPC8645EA 4×5MP Multi-sensor, LightHunter, IR 50m, IP67, 2.7-13.5mm","IPC8645EA-ADZKM-I1","XLR Security",1474],
  ["Camera","Uniview 4MP Eco Turret, IR+LED 30m, Fixed 2.8mm","EC-T4F28M-DL","XLR Security",102],
  ["Camera","Uniview ED-525B-WB Dual-lens Video Doorbell","ED-525B-WB","XLR Security",187],
  ["Recording","UniFi UNVR-G2 — 1U NVR, AI analytics, 4-bay, 30x4K","UNVR-G2","Ubiquiti CA",1005],
  ["Recording","UniFi UNVR-G2 Pro — 2U NVR, AI analytics, 8-bay, 50x4K / 100x HD","UBI-UNVR-G2-Pro","XLR Security",1579],
  ["Recording","UniFi ENVR — 3U NVR, 16-bay, 70x4K / 210x HD cameras","UBI-ENVR","XLR Security",2570],
  ["Recording","UniFi UNVR Pro 7-Bay — up to 60 days / 20x4K","UBI-UNVR-PRO","XLR Security",759],
  ["Recording","UniFi UCG Max 2TB — Cloud Gateway + NVR","UBI-UCG-Max-2TB","XLR Security",775],
  ["Recording","UniFi UCG Max 1TB — Cloud Gateway + NVR","UBI-UCG-Max-1TB","XLR Security",537],
  ["Recording","WD Purple 10TB Surveillance HDD","HDD-WD-PUR10TB","XLR Security",349],
  ["Recording","WD Purple 8TB Surveillance HDD","HDD-WD-PUR8TB","XLR Security",377],
  ["Recording","WD Purple 4TB Surveillance HDD","HDD-WD-PUR4TB","XLR Security",185],
  ["Recording","WD Purple 2TB Surveillance HDD","HDD-WD-PUR2TB","XLR Security",99],
  ["Recording","Toshiba 8TB HDD — Data Center + Video Surveillance","HDD-TOS-MG8TB","XLR Security",427],
  ["Recording","Toshiba 24TB HDD — Data Center + Video Surveillance","HDD-TOS-MG24TB","XLR Security",1027],
  ["Network","UniFi UDM Pro — OS Console + Security Gateway","UBI-UDM-PRO","Ubiquiti CA",545],
  ["Network","UniFi USW-48-POE — 48-port Layer 2 PoE+ 195W","UBI-USW-48-POE","Ubiquiti CA",846],
  ["Network","UniFi USW Pro Max 48-PoE 720W — Layer 3 PoE++","UBI-USW-Pro-Max-48-PoE","XLR Security",1961],
  ["Network","UniFi USW Pro Max 24-PoE 400W — Layer 3 PoE++","UBI-USW-Pro-Max-24-PoE","XLR Security",1253],
  ["Network","UniFi USW Pro Max 16-PoE 180W — Layer 3 PoE++","UBI-USW-Pro-Max-16-PoE","XLR Security",599],
  ["Network","UniFi USW Flex 2.5G 8-PoE (196W)","UBI-USW-Flex-2.5G-8-PoE","XLR Security",315],
  ["Network","UniFi USW 24-PoE — 24-port Layer 2 PoE","UBI-USW-24-POE","XLR Security",579],
  ["Network","UniFi USW Ultra 60W — 8-port GbE PoE","UBI-USW-Ultra-60W","XLR Security",237],
  ["Network","UniFi U7 Pro WiFi 7 AP — 6GHz, 2.5Gb uplink","UBI-U7-PRO-US","Ubiquiti CA",253],
  ["Network","UniFi U7 Pro XGS (Black) — 8-stream WiFi 7, 10G","UBI-U7-Pro-XGS-B","XLR Security",475],
  ["Network","UniFi Power Distribution Pro (USP-PDU-Pro)","USP-PDU-Pro","Ubiquiti CA",379],
  ["Network","UniFi U-PoE++ Adapter","UBI-U-PoE++","XLR Security",45],
  ["Network","Provo PN-D7028-24P-360 — 24-port GbE PoE+ 360W","PN-D7028-24P-360","XLR Security",257],
  ["Network","Reyee RG-EST310-V2 — P2P Wireless Bridge Pair","RG-EST310-V2","XLR Security",156],
  ["Access Control","UniFi EAH-8 Enterprise Access Hub — 8 doors","EAH-8","Ubiquiti CA",1379],
  ["Access Control","UniFi UA Hub — PoE++, DIN rail, 1 door","UBI-UA-HUB","XLR Security",301],
  ["Access Control","UniFi UA Hub Gate — Gate Hub","UBI-UA-Hub-Gate","XLR Security",454],
  ["Access Control","UniFi UA Ultra — Reader + hub, 1 door","UBI-UA-Ultra","XLR Security",180],
  ["Access Control","UniFi UA G3 Reader — NFC + Touch Pass","UA-G3-W","XLR Security",237],
  ["Access Control","UniFi UA G3 Pro (White) — Video intercom","UA-G3-PRO-W","Ubiquiti CA",520],
  ["Access Control","UniFi UA G3 Pro (Black) — Video intercom","UA-G3-PRO-B","XLR Security",552],
  ["Access Control","UniFi UA G3 Flex (White) — NFC keypad","UA-G3-Flex-W","XLR Security",287],
  ["Access Control","UniFi UA G3 Intercom — Indoor/outdoor terminal","UBI-UA-G3-Intercom","XLR Security",749],
  ["Access Control","UniFi UA Intercom Viewer — 5\" touch display","UBI-UA-INTERCOM-VIEWER","XLR Security",239],
  ["Access Control","UniFi UA Magnetic Lock 270KG — Fail-safe","UA-LOCK-MAGNETIC-270KG","Ubiquiti CA",199],
  ["Access Control","UniFi UACC Chime PoE","UBI-UACC-CHIME-POE","XLR Security",117],
  ["Access Control","UniFi UACC AC 210W Adapter","UBI-UACC-Adapter-AC-210W","XLR Security",135],
  ["Access Control","UniFi UP FloodLight — Smart floodlight","UBI-UP-FloodLight","XLR Security",153],
  ["Access Control","Akuvox AK-E12W — 2MP video intercom, NFC, Wi-Fi","AK-E12W","XLR Security",230],
  ["Cabling","Uniview Cat6 FT4 Riser (White), 1000ft","CAB-LC3100A-CMR-WH-IN","XLR Security",155],
  ["Cabling","XLR Cat6A FT4 Riser (White), 1000ft","XLR-CAB-C6A-FT4-1000W","XLR Security",270],
  ["Cabling","XLR Cat5E Direct Burial 1000ft, UV Resistant","XLR-CAB-C5-OUTDOOR","XLR Security",185],
  ["Cabling","CAT6 UTP CMR Riser Cable (per 1000ft)","CAT6-1000","Clever Cabling",85],
  ["Cabling","CAT6A UTP CMR Riser Cable (per 1000ft)","CAT6A-1000","Clever Cabling",145],
  ["Cabling","RJ45 CAT6 Connectors (bag of 50)","RJ45-50PK","Clever Cabling",12],
  ["Cabling","CAT6 Keystone Jack T568B (each)","KJ-CAT6","Clever Cabling",3.50],
  ["Cabling","Patch Panel 24-Port CAT6 1U","PP-24P-1U","Clever Cabling",38],
  ["Cabling","Patch Panel 48-Port CAT6 2U","PP-48P-2U","Clever Cabling",65],
  ["Fiber","OS2 Single-Mode Fiber 2-Strand (per 1000ft)","OS2-2ST-1000","Clever Cabling",145],
  ["Fiber","OM3 Multimode Fiber 2-Strand (per 1000ft)","OM3-2ST-1000","Clever Cabling",125],
  ["Fiber","LC-LC Single-Mode Patch — 3m","LC-LC-SM-3M","Clever Cabling",12],
  ["Fiber","12-Port Fiber Patch Panel 1U (LC)","FPP-12LC-1U","Clever Cabling",65],
  ["Fiber","SFP+ Module Single-Mode 10G","SFP-SM-10G","Clever Cabling",35],
  ["Fiber","SFP+ Module Multimode 10G","SFP-MM-10G","Clever Cabling",25],
  ["Infrastructure","CyberPower PR2200RT2UC — 2200VA/2200W Sine Wave 2U","CP-PR2200RT2UC","DeployDepot CA",2490],
  ["Infrastructure","CyberPower UPS 1500VA/1000W, 2U rack","UPS-CYBERPOWER-1500VA","XLR Security",485],
  ["Infrastructure","18U Floor-Standing Open Rack","RACK-18U-2P","Clever Cabling",285],
  ["Infrastructure","12U Wall-Mount Network Cabinet","CAB-12U-WM","Clever Cabling",195],
  ["Infrastructure","XLR 9U Network Cabinet, hinged glass door, 450mm deep","XLR-RACK-WM09-S","XLR Security",185],
  ["Infrastructure","XLR 4U Wall-Mount Network Cabinet","XLR-RACK-WM04","XLR Security",167],
  ["Infrastructure","Conduit 3/4\" EMT 10ft","EMT-0.75-10","Clever Cabling",12],
  ["Infrastructure","Conduit 1/2\" EMT 10ft","EMT-0.5-10","Clever Cabling",8],
  ["Infrastructure","J-Hook Cable Support 1\" (box of 25)","JH-1-25PK","Clever Cabling",45],
  ["Infrastructure","Surface Mount Box — Single Gang","SMB-1G","Clever Cabling",5],
  ["Infrastructure","Surface Mount Box — Double Gang","SMB-2G","Clever Cabling",8],
  ["Infrastructure","HDMI Extender over Cat5e/Cat6, 60m","XLR-ACC-HDMI-EXTENDER-60M","XLR Security",59],
  ["Mounting","Uniview TR-JB03-G-IN — Bracket IPC31x/IPC32x","TR-JB03-G-IN","XLR Security",11],
  ["Mounting","Uniview TR-JB03-H-IN — Bracket varifocal turrets","TR-JB03-H-IN","XLR Security",11],
  ["Mounting","Uniview TR-JB04-C-IN — Bracket IPC34x cameras","TR-JB04-C-IN","XLR Security",14],
  ["Mounting","Uniview TR-WM03-D-IN — Wall bracket domes/turrets","TR-WM03-D-IN","XLR Security",19],
  ["Mounting","Uniview TR-WE45-A — Extended wall bracket for PTZ cameras","TR-WE45-A","XLR Security",39],
  ["Mounting","Uniview TR-UC08-A — Corner mount for PTZ and dome cameras","TR-UC08-A","XLR Security",21],
  ["Mounting","UniFi UACC Camera Junction Box (Black) — Bullet/Dome/Turret","UBI-UACC-Camera-JB-B","XLR Security",79],
  ["Audio/AV","Uniview IPS302030-S — 30W Outdoor IP Speaker, PoE+","IPS302030-S","XLR Security",302],
  ["Audio/AV","Uniview MW-LC27 — 27\" LED Monitor, 1080p, HDMI+VGA","MW-LC27","XLR Security",173],
  ["Misc","24V AC Plug-in Transformer, 24VAC 40VA","XLR-PWR-24VAC","XLR Security",19.50],
];

const buildCat=raw=>raw.map((r,i)=>({id:i,cat:r[0],desc:r[1],part:r[2],sup:r[3],cost:r[4],url:UBI[r[2]]||r[5]||null}));

let _u=1;const uid=()=>"r"+(_u++);
const $c=n=>(typeof n==="number"&&isFinite(n))?new Intl.NumberFormat("en-CA",{style:"currency",currency:"CAD",minimumFractionDigits:2}).format(n):"$0.00";
const pct=n=>isFinite(n)?(n*100).toFixed(1)+"%":"0.0%";
const nv=v=>parseFloat(v)||0;
const IS=(a)=>({width:"100%",border:"1px solid #dde1e8",borderRadius:3,padding:"3px 5px",fontSize:11,background:"white",outline:"none",textAlign:a||"left",boxSizing:"border-box",fontFamily:"inherit"});

function TIn({v,on,ph}){return el("input",{value:v,onChange:e=>on(e.target.value),placeholder:ph||"",style:IS(),onFocus:e=>(e.target.style.borderColor="#2563EB"),onBlur:e=>(e.target.style.borderColor="#dde1e8")})}
function NIn({v,on,ph}){return el("input",{type:"number",value:v||"",onChange:e=>on(nv(e.target.value)),placeholder:ph||"0",min:"0",step:"any",style:IS("right"),onFocus:e=>(e.target.style.borderColor="#2563EB"),onBlur:e=>(e.target.style.borderColor="#dde1e8")})}

function LR({r,u,d}){const a=r.dayRate*r.days;return el("tr",{style:{borderBottom:"1px solid #F1F5F9"}},
  el("td",{style:{padding:"3px 4px",width:22}},el("button",{onClick:()=>d(r.id),style:{border:"none",background:"none",cursor:"pointer",color:"#CBD5E1",fontSize:14,padding:0}},"×")),
  el("td",{style:{padding:"2px 4px"}},el(TIn,{v:r.desc,on:v=>u(r.id,"desc",v),ph:"Scope description"})),
  el("td",{style:{padding:"2px 4px",width:108}},el("select",{value:r.type,onChange:e=>u(r.id,"type",e.target.value),style:{...IS(),fontSize:11}},["Own Staff","Subcontractor","Project Manager","Other"].map(t=>el("option",{key:t},t)))),
  el("td",{style:{padding:"2px 4px",width:82}},el(NIn,{v:r.dayRate,on:v=>u(r.id,"dayRate",v),ph:"850"})),
  el("td",{style:{padding:"2px 4px",width:58}},el(NIn,{v:r.days,on:v=>u(r.id,"days",v),ph:"1"})),
  el("td",{style:{padding:"2px 8px 2px 4px",width:86,textAlign:"right",fontSize:11,fontWeight:500,color:a>0?"#1E293B":"#CBD5E1"}},$c(a))
)}

function ER({r,u,d}){const a=r.qty*r.cost,url=UBI[r.part]||null;return el("tr",{style:{borderBottom:"1px solid #F1F5F9"}},
  el("td",{style:{padding:"3px 4px",width:22}},el("button",{onClick:()=>d(r.id),style:{border:"none",background:"none",cursor:"pointer",color:"#CBD5E1",fontSize:14,padding:0}},"×")),
  el("td",{style:{padding:"2px 4px"}},el("div",{style:{display:"flex",alignItems:"center",gap:3}},el(TIn,{v:r.desc,on:v=>u(r.id,"desc",v),ph:"Item description"}),url&&el("a",{href:url,target:"_blank",rel:"noreferrer",style:{color:"#2563EB",fontSize:10,flexShrink:0}},"↗"))),
  el("td",{style:{padding:"2px 4px",width:100}},el(TIn,{v:r.sup,on:v=>u(r.id,"sup",v),ph:"Supplier"})),
  el("td",{style:{padding:"2px 4px",width:90}},el(TIn,{v:r.part,on:v=>u(r.id,"part",v),ph:"Part #"})),
  el("td",{style:{padding:"2px 4px",width:44}},el(NIn,{v:r.qty,on:v=>u(r.id,"qty",v),ph:"1"})),
  el("td",{style:{padding:"2px 4px",width:78}},el(NIn,{v:r.cost,on:v=>u(r.id,"cost",v),ph:"0.00"})),
  el("td",{style:{padding:"2px 8px 2px 4px",width:86,textAlign:"right",fontSize:11,fontWeight:500,color:a>0?"#1E293B":"#CBD5E1"}},$c(a))
)}

function AR({r,u,d}){return el("tr",{style:{borderBottom:"1px solid #F1F5F9"}},
  el("td",{style:{padding:"3px 4px",width:22}},el("button",{onClick:()=>d(r.id),style:{border:"none",background:"none",cursor:"pointer",color:"#CBD5E1",fontSize:14,padding:0}},"×")),
  el("td",{style:{padding:"2px 4px"}},el(TIn,{v:r.desc,on:v=>u(r.id,"desc",v),ph:"Description"})),
  el("td",{style:{padding:"2px 4px"}},el(TIn,{v:r.notes,on:v=>u(r.id,"notes",v),ph:"Notes"})),
  el("td",{style:{padding:"2px 8px 2px 4px",width:96}},el(NIn,{v:r.amount,on:v=>u(r.id,"amount",v),ph:"0.00"}))
)}

function CH({cols}){return el("thead",null,el("tr",{style:{background:"#F8FAFC"}},el("th",{style:{width:22,padding:"3px 4px"}}),cols.map((c,i)=>el("th",{key:i,style:{padding:"4px",fontSize:10,fontWeight:600,color:"#94A3B8",textAlign:c.r?"right":"left",width:c.w,whiteSpace:"nowrap"}},c.label))))}

function Sec({title,letter,accent,count,sub,open,toggle,onAdd,cols,children}){
  return el("div",{style:{marginBottom:8,border:"1px solid #E2E8F0",borderRadius:6,overflow:"hidden"}},
    el("div",{onClick:toggle,style:{display:"flex",alignItems:"center",gap:8,padding:"7px 10px",background:accent,cursor:"pointer",userSelect:"none"}},
      el("span",{style:{width:20,height:20,borderRadius:"50%",background:"rgba(255,255,255,0.18)",display:"flex",alignItems:"center",justifyContent:"center",fontSize:10,fontWeight:700,color:"#fff",flexShrink:0}},letter),
      el("span",{style:{fontWeight:600,fontSize:12,color:"#fff",flex:1,letterSpacing:0.3}},title),
      count>0&&el("span",{style:{fontSize:10.5,color:"rgba(255,255,255,0.75)",marginRight:2}},count+" item"+(count!==1?"s":"")+" · "+$c(sub)),
      el("span",{style:{color:"rgba(255,255,255,0.8)",fontSize:14}},open?"▲":"▼")
    ),
    open&&el("div",{style:{background:"#fff"}},
      count>0&&el("table",{style:{width:"100%",borderCollapse:"collapse",tableLayout:"fixed"}},el(CH,{cols}),el("tbody",null,...(Array.isArray(children)?children:[children]))),
      el("div",{style:{padding:"5px 8px"}},el("button",{onClick:onAdd,style:{display:"flex",alignItems:"center",gap:4,border:"1px dashed #CBD5E1",borderRadius:4,padding:"4px 10px",background:"white",cursor:"pointer",fontSize:11,color:"#64748B"}},"+ Add row"))
    )
  )
}

function SL({label,value,bold,hl,top,size}){
  return el("div",{style:{display:"flex",justifyContent:"space-between",padding:bold?"5px 12px":"3px 12px",background:hl||"transparent",borderTop:top?"1px solid #E2E8F0":undefined,fontWeight:bold?600:400,fontSize:size||11.5,color:"#475569"}},
    el("span",null,label),el("span",null,value))
}

function App(){
  const[proj,setProj]=useState({client:"",address:"",ref:"",date:new Date().toISOString().slice(0,10),type:"",estimator:""});
  const[labour,setLabour]=useState([]);
  const[equip,setEquip]=useState([]);
  const[matl,setMatl]=useState([]);
  const[addl,setAddl]=useState([]);
  const[cfg,setCfg]=useState({overhead:15,margin:30});
  const[catalog,setCatalog]=useState(()=>buildCat(DEF_CAT));
  const[search,setSearch]=useState("");
  const[catF,setCatF]=useState("All");
  const[open,setOpen]=useState({A:true,B:true,C:true,D:false});
  const[cfgOpen,setCfgOpen]=useState(false);
  const[flash,setFlash]=useState(null);
  const[toast,setToast]=useState(null);
  const catRef=useRef(),projRef=useRef();

  const ALL_CATS=useMemo(()=>["All",...new Set(catalog.map(c=>c.cat))],[catalog]);
  const T=useMemo(()=>{
    const lS=labour.reduce((s,r)=>s+r.dayRate*r.days,0);
    const eS=equip.reduce((s,r)=>s+r.qty*r.cost,0);
    const mS=matl.reduce((s,r)=>s+r.qty*r.cost,0);
    const aS=addl.reduce((s,r)=>s+r.amount,0);
    const direct=lS+eS+mS+aS,oh=direct*cfg.overhead/100,sub=direct+oh;
    const m=cfg.margin/100,quote=m<1?sub/(1-m):sub;
    const profit=quote-sub,hst=quote*0.13,inv=quote+hst;
    return{lS,eS,mS,aS,direct,oh,sub,quote,profit,hst,inv,mgPct:quote>0?profit/quote:0};
  },[labour,equip,matl,addl,cfg]);

  const filtered=useMemo(()=>{
    const q=search.toLowerCase();
    return catalog.filter(it=>(catF==="All"||it.cat===catF)&&(!q||it.desc.toLowerCase().includes(q)||it.part.toLowerCase().includes(q)||it.sup.toLowerCase().includes(q)));
  },[catalog,search,catF]);

  const showToast=msg=>{setToast(msg);setTimeout(()=>setToast(null),3000)};

  const importCatalog=e=>{
    const file=e.target.files[0];if(!file)return;
    const r=new FileReader();
    r.onload=ev=>{
      try{
        const items=JSON.parse(ev.target.result);
        if(!Array.isArray(items))throw new Error();
        const existing=new Set(catalog.map(c=>c.part));
        const next=items.filter(it=>it.cat&&it.desc&&it.part&&it.cost).map((it,i)=>({id:catalog.length+i+Date.now(),cat:it.cat,desc:it.desc,part:it.part,sup:it.sup||"Custom",cost:parseFloat(it.cost)||0,url:it.url||UBI[it.part]||null}));
        const added=next.filter(it=>!existing.has(it.part));
        const upd=next.filter(it=>existing.has(it.part));
        setCatalog(prev=>{const m=prev.map(c=>{const u=upd.find(x=>x.part===c.part);return u?{...c,...u}:c});return[...m,...added]});
        showToast("✓ Catalog: "+added.length+" added, "+upd.length+" updated");
      }catch{showToast("⚠ Invalid JSON. Format: [{cat,desc,part,sup,cost,url?}]")}
    };
    r.readAsText(file);e.target.value="";
  };

  const exportProject=()=>{
    const data={proj,labour,equip,matl,addl,cfg,savedAt:new Date().toISOString()};
    const blob=new Blob([JSON.stringify(data,null,2)],{type:"application/json"});
    const url=URL.createObjectURL(blob);
    const a=document.createElement("a");
    a.href=url;a.download="citadel-"+(proj.client||"project").replace(/\s+/g,"-").toLowerCase()+"-"+(proj.ref||"est")+".json";
    a.click();URL.revokeObjectURL(url);
  };

  const importProject=e=>{
    const file=e.target.files[0];if(!file)return;
    const r=new FileReader();
    r.onload=ev=>{
      try{
        const s=JSON.parse(ev.target.result);
        setProj(s.proj||{});setLabour(s.labour||[]);setEquip(s.equip||[]);setMatl(s.matl||[]);setAddl(s.addl||[]);setCfg(s.cfg||{overhead:15,margin:30});
        showToast("✓ Loaded: "+(s.proj?.client||file.name));
      }catch{showToast("⚠ Invalid project file.")}
    };
    r.readAsText(file);e.target.value="";
  };

  const uL=(id,k,v)=>setLabour(p=>p.map(r=>r.id===id?{...r,[k]:v}:r));
  const uE=(id,k,v)=>setEquip(p=>p.map(r=>r.id===id?{...r,[k]:v}:r));
  const uM=(id,k,v)=>setMatl(p=>p.map(r=>r.id===id?{...r,[k]:v}:r));
  const uA=(id,k,v)=>setAddl(p=>p.map(r=>r.id===id?{...r,[k]:v}:r));
  const dL=id=>setLabour(p=>p.filter(r=>r.id!==id));
  const dE=id=>setEquip(p=>p.filter(r=>r.id!==id));
  const dM=id=>setMatl(p=>p.filter(r=>r.id!==id));
  const dA=id=>setAddl(p=>p.filter(r=>r.id!==id));
  const aL=()=>{setLabour(p=>[...p,{id:uid(),desc:"",type:"Subcontractor",dayRate:850,days:1}]);setOpen(o=>({...o,A:true}))};
  const aE=()=>{setEquip(p=>[...p,{id:uid(),desc:"",sup:"XLR Security",part:"",qty:1,cost:0}]);setOpen(o=>({...o,B:true}))};
  const aM=()=>{setMatl(p=>[...p,{id:uid(),desc:"",sup:"Clever Cabling",part:"",qty:1,cost:0}]);setOpen(o=>({...o,C:true}))};
  const aAd=()=>{setAddl(p=>[...p,{id:uid(),desc:"",notes:"",amount:0}]);setOpen(o=>({...o,D:true}))};
  const catAdd=(item,sec)=>{
    const row={id:uid(),desc:item.desc,sup:item.sup,part:item.part,qty:1,cost:item.cost};
    if(sec==="eq"){setEquip(p=>[...p,row]);setOpen(o=>({...o,B:true}))}else{setMatl(p=>[...p,row]);setOpen(o=>({...o,C:true}))}
    setFlash(item.id+sec);setTimeout(()=>setFlash(null),1200);
  };
  const tog=k=>setOpen(o=>({...o,[k]:!o[k]}));
  const setP=(k,v)=>setProj(p=>({...p,[k]:v}));
  const mgColor=T.mgPct>=0.25?"#16A34A":T.mgPct>=0.10?"#D97706":"#DC2626";

  const PI=({label,k,ph,w})=>el("div",{style:{flex:w||"1"}},
    el("div",{style:{fontSize:9,fontWeight:600,color:"rgba(255,255,255,0.5)",marginBottom:2,letterSpacing:0.5}},label),
    el("input",{value:proj[k]||"",onChange:e=>setP(k,e.target.value),placeholder:ph||"",style:{background:"rgba(255,255,255,0.1)",border:"1px solid rgba(255,255,255,0.18)",borderRadius:3,padding:"3px 7px",fontSize:11,color:"white",width:"100%",outline:"none",boxSizing:"border-box",fontFamily:"inherit"}})
  );

  const lCols=[{label:"Scope"},{label:"Type",w:108},{label:"Day Rate ($)",w:82,r:true},{label:"Days",w:58,r:true},{label:"Amount",w:86,r:true}];
  const eCols=[{label:"Description / Model"},{label:"Supplier",w:100},{label:"Part #",w:90},{label:"Qty",w:44,r:true},{label:"Unit Cost",w:78,r:true},{label:"Amount",w:86,r:true}];
  const aCols=[{label:"Description"},{label:"Notes / Reference"},{label:"Amount",w:96,r:true}];

  const Btn=(props)=>el("button",{...props,style:{display:"flex",alignItems:"center",justifyContent:"center",border:"1px solid rgba(255,255,255,0.18)",borderRadius:4,padding:"5px 8px",color:"white",cursor:"pointer",background:"rgba(255,255,255,0.1)",fontFamily:"inherit",...(props.style||{})}});

  return el("div",{style:{display:"flex",height:"100vh",fontFamily:"system-ui,sans-serif",background:"#F1F5F9",overflow:"hidden",fontSize:12,position:"relative"}},

    toast&&el("div",{style:{position:"fixed",top:16,left:"50%",transform:"translateX(-50%)",background:"#1C3557",color:"white",padding:"8px 16px",borderRadius:6,fontSize:12,fontWeight:500,zIndex:9999,boxShadow:"0 4px 12px rgba(0,0,0,0.2)",whiteSpace:"nowrap"}},toast),

    el("input",{ref:catRef,type:"file",accept:".json",onChange:importCatalog,style:{display:"none"}}),
    el("input",{ref:projRef,type:"file",accept:".json",onChange:importProject,style:{display:"none"}}),

    // SIDEBAR
    el("div",{style:{width:268,flexShrink:0,display:"flex",flexDirection:"column",background:"#1C3557",overflow:"hidden",borderRight:"1px solid rgba(255,255,255,0.08)"}},
      el("div",{style:{padding:"10px 10px 7px",borderBottom:"1px solid rgba(255,255,255,0.08)"}},
        el("div",{style:{fontSize:10,fontWeight:700,color:"rgba(255,255,255,0.4)",letterSpacing:1.2,marginBottom:7}},"EQUIPMENT CATALOG"),
        el("input",{value:search,onChange:e=>setSearch(e.target.value),placeholder:"Search items or part #...",style:{width:"100%",background:"rgba(255,255,255,0.09)",border:"1px solid rgba(255,255,255,0.13)",borderRadius:4,padding:"5px 8px",fontSize:11,color:"white",outline:"none",boxSizing:"border-box",fontFamily:"inherit",marginBottom:6}}),
        el("button",{onClick:()=>catRef.current.click(),style:{width:"100%",display:"flex",alignItems:"center",justifyContent:"center",gap:5,background:"rgba(255,255,255,0.07)",border:"1px dashed rgba(255,255,255,0.2)",borderRadius:4,padding:"5px 0",cursor:"pointer",fontSize:10,color:"rgba(255,255,255,0.55)",fontFamily:"inherit"}},
          "⊕ Import catalog items (JSON)")
      ),
      el("div",{style:{display:"flex",overflowX:"auto",padding:"5px 8px",gap:4,flexShrink:0,borderBottom:"1px solid rgba(255,255,255,0.08)"}},
        ALL_CATS.map(c=>el("button",{key:c,onClick:()=>setCatF(c),style:{flexShrink:0,padding:"2px 8px",borderRadius:10,fontSize:10,cursor:"pointer",border:"none",fontWeight:catF===c?600:400,fontFamily:"inherit",background:catF===c?"#EA580C":"rgba(255,255,255,0.09)",color:catF===c?"white":"rgba(255,255,255,0.55)"}},c))
      ),
      el("div",{style:{flex:1,overflowY:"auto",padding:"2px 0"}},
        filtered.length===0&&el("div",{style:{padding:"20px",textAlign:"center",color:"rgba(255,255,255,0.25)",fontSize:11}},"No items found"),
        filtered.map(item=>{
          const isEq=flash===item.id+"eq",isMt=flash===item.id+"mt";
          return el("div",{key:item.id,style:{padding:"6px 10px",borderBottom:"1px solid rgba(255,255,255,0.045)"},
            onMouseEnter:e=>(e.currentTarget.style.background="rgba(255,255,255,0.055)"),
            onMouseLeave:e=>(e.currentTarget.style.background="transparent")},
            el("div",{style:{display:"flex",alignItems:"flex-start",gap:4}},
              el("div",{style:{flex:1,fontSize:10.5,color:"rgba(255,255,255,0.88)",lineHeight:1.35,marginBottom:2}},item.desc),
              item.url&&el("a",{href:item.url,target:"_blank",rel:"noreferrer",style:{color:"#93C5FD",flexShrink:0,marginTop:1,fontSize:11}},"↗")
            ),
            el("div",{style:{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:4}},
              el("span",{style:{fontSize:9.5,color:"rgba(255,255,255,0.3)"}},item.part+" · "+item.sup),
              el("span",{style:{fontSize:10.5,fontWeight:600,color:"#FDE68A"}},$c(item.cost))
            ),
            el("div",{style:{display:"flex",gap:4}},
              el("button",{onClick:()=>catAdd(item,"eq"),style:{flex:1,padding:"2px 0",border:"1px solid "+(isEq?"#3B82F6":"rgba(59,130,246,0.4)"),borderRadius:3,fontSize:9.5,cursor:"pointer",fontFamily:"inherit",background:isEq?"rgba(59,130,246,0.4)":"rgba(59,130,246,0.15)",color:isEq?"#BFDBFE":"#93C5FD"}},isEq?"✓ Added":"+ Equipment"),
              el("button",{onClick:()=>catAdd(item,"mt"),style:{flex:1,padding:"2px 0",border:"1px solid "+(isMt?"#10B981":"rgba(16,185,129,0.35)"),borderRadius:3,fontSize:9.5,cursor:"pointer",fontFamily:"inherit",background:isMt?"rgba(16,185,129,0.35)":"rgba(16,185,129,0.12)",color:isMt?"#A7F3D0":"#86EFAC"}},isMt?"✓ Added":"+ Materials")
            )
          )
        })
      ),
      el("div",{style:{padding:"5px 10px",borderTop:"1px solid rgba(255,255,255,0.08)",fontSize:9,color:"rgba(255,255,255,0.2)",textAlign:"center"}},filtered.length+"/"+catalog.length+" items · Ubiquiti CA · XLR · Clever Cabling · DeployDepot")
    ),

    // MAIN
    el("div",{style:{flex:1,display:"flex",flexDirection:"column",overflow:"hidden",minWidth:0}},

      // Header
      el("div",{style:{background:"#1C3557",padding:"8px 12px",display:"flex",alignItems:"flex-end",gap:6,flexShrink:0}},
        el("div",{style:{fontSize:12,fontWeight:700,color:"white",lineHeight:"26px",whiteSpace:"nowrap",letterSpacing:0.5}},"CITADEL ·"),
        el(PI,{label:"CLIENT",k:"client",ph:"Client name",w:"1.8"}),
        el(PI,{label:"SITE ADDRESS",k:"address",ph:"Site address",w:"2"}),
        el(PI,{label:"REF #",k:"ref",ph:"EST-001",w:"0.8"}),
        el(PI,{label:"DATE",k:"date",w:"1.1"}),
        el(PI,{label:"TYPE",k:"type",ph:"CCTV / AC / Net",w:"1.2"}),
        el(PI,{label:"ESTIMATOR",k:"estimator",ph:"Name",w:"1"}),
        el(Btn,{onClick:()=>projRef.current.click(),title:"Load project from JSON",style:{fontSize:11}},
          el("span",null,"📂")),
        el(Btn,{onClick:exportProject,title:"Save project as JSON",style:{fontSize:11}},
          el("span",null,"💾")),
        el(Btn,{onClick:()=>setCfgOpen(o=>!o),title:"Settings",style:{background:cfgOpen?"rgba(234,88,12,0.8)":"rgba(255,255,255,0.1)"}},
          el("span",{style:{fontSize:13}},"⚙"))
      ),

      cfgOpen&&el("div",{style:{background:"#253E5C",padding:"8px 14px",display:"flex",alignItems:"center",gap:20,flexShrink:0,borderBottom:"1px solid rgba(255,255,255,0.08)"}},
        el("span",{style:{fontSize:10,fontWeight:700,color:"rgba(255,255,255,0.45)",letterSpacing:1}},"SETTINGS"),
        ...[["Overhead %","overhead",0,50],["Target Margin %","margin",0,60]].map(([label,key,mn,mx])=>
          el("div",{key,style:{display:"flex",alignItems:"center",gap:7}},
            el("span",{style:{fontSize:11,color:"rgba(255,255,255,0.65)",whiteSpace:"nowrap"}},label),
            el("input",{type:"range",min:mn,max:mx,step:1,value:cfg[key],onChange:e=>setCfg(p=>({...p,[key]:+e.target.value})),style:{width:90,accentColor:"#EA580C"}}),
            el("span",{style:{fontSize:12,fontWeight:700,color:"#FDE68A",minWidth:34}},cfg[key]+"%")
          )
        ),
        el("span",{style:{fontSize:11,color:"rgba(255,255,255,0.4)"}},"HST: 13% (Ontario, fixed)")
      ),

      el("div",{style:{flex:1,overflow:"hidden",display:"flex",gap:10,padding:"10px",minHeight:0}},
        el("div",{style:{flex:1,overflowY:"auto",minWidth:0}},
          el(Sec,{title:"LABOUR & CONTRACTOR COSTS",letter:"A",accent:"#2563EB",count:labour.length,sub:T.lS,open:open.A,toggle:()=>tog("A"),onAdd:aL,cols:lCols},labour.map(r=>el(LR,{key:r.id,r,u:uL,d:dL}))),
          el(Sec,{title:"EQUIPMENT COSTS",letter:"B",accent:"#0F766E",count:equip.length,sub:T.eS,open:open.B,toggle:()=>tog("B"),onAdd:aE,cols:eCols},equip.map(r=>el(ER,{key:r.id,r,u:uE,d:dE}))),
          el(Sec,{title:"MATERIALS & CABLING",letter:"C",accent:"#6D28D9",count:matl.length,sub:T.mS,open:open.C,toggle:()=>tog("C"),onAdd:aM,cols:eCols},matl.map(r=>el(ER,{key:r.id,r,u:uM,d:dM}))),
          el(Sec,{title:"ADDITIONAL & MISCELLANEOUS",letter:"D",accent:"#B45309",count:addl.length,sub:T.aS,open:open.D,toggle:()=>tog("D"),onAdd:aAd,cols:aCols},addl.map(r=>el(AR,{key:r.id,r,u:uA,d:dA})))
        ),

        el("div",{style:{width:212,flexShrink:0,overflowY:"auto"}},
          el("div",{style:{position:"sticky",top:0,border:"1px solid #E2E8F0",borderRadius:6,overflow:"hidden",background:"white"}},
            el("div",{style:{background:"#1C3557",padding:"8px 12px"}},
              el("div",{style:{fontSize:10,fontWeight:700,color:"white",letterSpacing:0.8}},"COST SUMMARY"),
              proj.client&&el("div",{style:{fontSize:9,color:"rgba(255,255,255,0.4)",marginTop:1}},proj.client+" · "+proj.ref)
            ),
            el(SL,{label:"Labour",value:$c(T.lS)}),
            el(SL,{label:"Equipment",value:$c(T.eS)}),
            el(SL,{label:"Materials / Cabling",value:$c(T.mS)}),
            el(SL,{label:"Additional",value:$c(T.aS)}),
            el(SL,{label:"TOTAL DIRECT COSTS",value:$c(T.direct),bold:true,hl:"#F8FAFC",top:true}),
            el("div",{style:{padding:"2px 12px",borderTop:"1px solid #F8FAFC",display:"flex",justifyContent:"space-between",fontSize:11,color:"#94A3B8"}},el("span",null,"Overhead ("+cfg.overhead+"%)"),el("span",null,$c(T.oh))),
            el(SL,{label:"Subtotal (cost + overhead)",value:$c(T.sub),bold:true,hl:"#F8FAFC",top:true}),
            el("div",{style:{padding:"3px 12px 1px",fontSize:10,color:"#94A3B8"}},"Target margin: "+cfg.margin+"%"),
            el(SL,{label:"QUOTE PRICE (excl. HST)",value:$c(T.quote),bold:true,hl:"#EFF6FF",top:true,size:12}),
            el("div",{style:{padding:"2px 12px 4px",borderBottom:"1px solid #F1F5F9"}},
              el("div",{style:{display:"flex",justifyContent:"space-between",fontSize:11,color:"#64748B"}},el("span",null,"Gross Profit"),el("span",{style:{color:T.profit>=0?"#16A34A":"#DC2626",fontWeight:500}},$c(T.profit))),
              el("div",{style:{display:"flex",justifyContent:"space-between",fontSize:11}},el("span",{style:{color:"#64748B"}},"Gross Margin"),el("span",{style:{color:mgColor,fontWeight:600}},pct(T.mgPct)))
            ),
            el("div",{style:{display:"flex",justifyContent:"space-between",padding:"3px 12px",fontSize:11,color:"#64748B",borderBottom:"1px solid #F1F5F9"}},el("span",null,"HST (13%)"),el("span",null,$c(T.hst))),
            el("div",{style:{background:"#EA580C",padding:"10px 12px"}},
              el("div",{style:{fontSize:9,fontWeight:700,color:"rgba(255,255,255,0.75)",letterSpacing:0.8,marginBottom:2}},"TOTAL INVOICE AMOUNT"),
              el("div",{style:{fontSize:22,fontWeight:700,color:"white",lineHeight:1.1}},$c(T.inv)),
              el("div",{style:{fontSize:10,color:"rgba(255,255,255,0.65)",marginTop:2}},"incl. HST")
            ),
            el("div",{style:{padding:"6px 8px",display:"flex",gap:4}},
              el("button",{onClick:()=>projRef.current.click(),style:{flex:1,display:"flex",alignItems:"center",justifyContent:"center",gap:3,border:"1px solid #E2E8F0",borderRadius:3,padding:"4px 0",background:"white",cursor:"pointer",fontSize:10,color:"#475569",fontFamily:"inherit"}},
                "📂 Load"),
              el("button",{onClick:exportProject,style:{flex:1,display:"flex",alignItems:"center",justifyContent:"center",gap:3,border:"1px solid #E2E8F0",borderRadius:3,padding:"4px 0",background:"white",cursor:"pointer",fontSize:10,color:"#475569",fontFamily:"inherit"}},
                "💾 Save JSON")
            )
          )
        )
      )
    )
  )
}

// Fix for useMemo dependency syntax error above
ReactDOM.render(React.createElement(App),document.getElementById("root"));
</script>
</body>
</html>
