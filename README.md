<!DOCTYPE html><html lang="th"><head><meta charset="utf-8"/><title>Job Sheet v12.3.7-min</title><meta name="viewport" content="width=device-width,initial-scale=1"/><script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script><script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script><style>
:root{--a4-w:842px;--a4-h:595px}body{font-family:Helvetica,Arial,sans-serif;background:#f6f6f6;margin:16px}h1{text-align:center;font-size:20px;margin-bottom:8px}textarea{width:100%;height:200px;font-family:monospace;padding:8px;box-sizing:border-box}button{padding:8px 12px;margin:4px;font-size:15px;cursor:pointer}.controls{text-align:center;margin:10px 0}#notice{text-align:center;font-weight:bold;margin-top:8px}#preview-container{display:flex;flex-direction:column;gap:20px;margin-top:10px}.page-wrap{width:var(--a4-w);height:var(--a4-h);margin:0 auto;background:white;display:grid;grid-template-columns:repeat(2,1fr);grid-template-rows:repeat(2,1fr);box-shadow:0 0 6px rgba(0,0,0,.15);position:relative}.slot{box-sizing:border-box;padding:24px;display:flex;flex-direction:column;justify-content:space-between;font-weight:700;position:relative}.line-top{display:flex;justify-content:space-between;font-size:24px;font-weight:700}.line-top>div:first-child{font-size:36px;font-weight:900}.passenger{text-align:center;font-size:36px;font-weight:700;margin-top:32px;margin-bottom:16px;display:-webkit-box;-webkit-line-clamp:2;-webkit-box-orient:vertical;overflow:hidden;word-break:break-word}.route{text-align:center;font-size:24px;font-weight:700;margin:0;display:flex;justify-content:center;align-items:center;flex-grow:1;word-break:break-word}.code{position:absolute;right:24px;bottom:18px;font-size:12px;font-weight:400}
</style></head><body>
<h1>ใบงานสแตนบาย v1ล่าสุด</h1>
<textarea id="input" placeholder="วางหลายงานที่นี่..."></textarea>
<div class="controls">
<button id="btnPreview">ตัวอย่าง</button>
<button id="btnPDF">ดาวน์โหลด PDF</button>
<button id="btnSharePDF">📤 แชร์ PDF</button>
<button id="btnClear">ลบ</button>
</div>
<div id="notice"></div>
<div id="preview-container"></div>
<script>
(function(){
function splitJobs(t){let b=[],c=[];for(const l of t.split(/\r?\n/).map(s=>s.trim())){if(/^[A-Z]\d{3,5}-\d{1,2}/i.test(l)){if(c.length)b.push(c.join("\n"));c=[l]}else if(l)c.push(l)}if(c.length)b.push(c.join("\n"));return b}
function getVal(t,k){const m=t.match(new RegExp("【"+k+"】([^\\n]*)"));return m?m[1].trim():""}
function getPickup(t){for(const k of["接รับ","接","รับ","Pickup"]){const v=getVal(t,k);if(v)return v}return""}
function getDrop(t){for(const k of["送ส่ง","送","ส่ง","Dropoff"]){const v=getVal(t,k);if(v)return v}return""}
function shortName(n){if(!n)return"";n=n.split("(")[0].split(")")[0].trim();const w=n.replace(/[^\wก-๙\s]/g," ").trim().split(/\s+/);return w.slice(0,3).join(" ")}
function cleanRoute(p,d){return shortName(p||"รับ?")+" → "+shortName(d||"ส่ง?")}
function parseJob(b){const j=b.match(/^[A-Z]\d{3,5}-\d{1,2}/i)?.[0]||"";let cF=getVal(b,"code")||getVal(b,"Code")||getVal(b,"CODE")||getVal(b,"接驳编码code")||"";let cM=cF.match(/[A-Z0-9]+/i);return{jobcode:j,date:getVal(b,"日期วันที่"),time:getVal(b,"时间เวลา"),name:getVal(b,"姓名ชื่อ")||getVal(b,"姓名"),route:cleanRoute(getPickup(b),getDrop(b)),code:cM?cM[0]:""}} 
function getJobs(){const r=document.getElementById("input").value.trim();return r?splitJobs(r).map(parseJob):[]}
function makePage(j){const w=document.createElement("div");w.className="page-wrap";for(let i=0;i<4;i++){const j2=j[i];const s=document.createElement("div");s.className="slot";s.innerHTML=`<div class="line-top"><div>${j2?j2.jobcode:""}</div><div>${j2?j2.time:""}</div><div>${j2?j2.date:""}</div></div><div class="passenger">${j2?j2.name:""}</div><div class="route">${j2?j2.route:""}</div><div class="code">${j2?j2.code:""}</div>`;w.appendChild(s)}return w}
function fillPreview(){const w=document.getElementById("preview-container");w.innerHTML="";const j=getJobs();if(!j.length)return;document.getElementById("notice").textContent=`รวม ${j.length} งาน`;for(let i=0;i<j.length;i+=4)w.appendChild(makePage(j.slice(i,i+4)))}
async function buildPDF(){const {jsPDF}=window.jspdf;const j=getJobs();if(!j.length){alert("ไม่มีงาน");return null}const pdf=new jsPDF({orientation:"landscape",unit:"pt",format:[842,595]});const p=document.querySelectorAll(".page-wrap");for(let i=0;i<p.length;i++){const c=await html2canvas(p[i],{scale:2});pdf.addImage(c.toDataURL("image/png"),"PNG",0,0,842,595);if(i<p.length-1)pdf.addPage()}return pdf}
function getFileName(){const r=document.getElementById("input").value.trim();const m=r.match(/^([A-Z]\d{3,5})-/i);return m?m[1]:"Job_Sheet"}
async function downloadPDF(){const pdf=await buildPDF();if(!pdf)return;pdf.save(getFileName()+".pdf")}
async function sharePDF(){const pdf=await buildPDF();if(!pdf)return;const blob=pdf.output("blob");const fileName=getFileName()+".pdf";const file=new File([blob],fileName,{type:"application/pdf"});if(navigator.share&&navigator.canShare&&navigator.canShare({files:[file]})){try{await navigator.share({files:[file],title:fileName,text:"ไฟล์ใบงานล่าสุด"});return}catch(e){}}pdf.save(fileName)}
document.getElementById("btnPreview").onclick=fillPreview;
document.getElementById("btnPDF").onclick=downloadPDF;
document.getElementById("btnSharePDF").onclick=sharePDF;
document.getElementById("btnClear").onclick=()=>{document.getElementById("input").value="";fillPreview()};
})();
</script>
</body></html>
