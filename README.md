<!doctype html>
<html lang="ko">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>AAC 음성출력 도구</title>

<style>
body {
    margin:0;
    font-family:'Noto Sans KR',Arial,sans-serif;
    background:#f0f2f5;
}

.top {
    display:flex;
    justify-content:center;
    align-items:center;
    padding:16px;
    background:linear-gradient(90deg,#00c6ff,#0072ff);
    box-shadow:0 2px 6px rgba(0,0,0,0.2);
}

.btn {
    padding:12px 20px;
    font-size:18px;
    border-radius:12px;
    border:none;
    background:white;
    color:#0072ff;
    font-weight:bold;
    cursor:pointer;
}

.btn.primary {
    background:#0072ff;
    color:white;
    font-size:14px;
    padding:6px 12px;
}

.yes-no {
    display:flex;
    gap:24px;
    justify-content:center;
    flex:1;
}

.section {
    background:white;
    padding:20px;
    margin:20px;
    border-radius:16px;
    box-shadow:0 2px 8px rgba(0,0,0,0.1);
}

/* 이미지 배경판 */
.gallery {
    display:grid;
    grid-template-columns:repeat(5,1fr);
    gap:12px;
    margin-top:12px;
}

.thumb {
    width:100%;
    padding-top:100%;
    position:relative;
    border-radius:12px;
    overflow:hidden;
    cursor:grab;
    transition:transform 0.15s;
    border:2px solid #e0e0e0;

    touch-action:none;
    user-select:none;
    -webkit-user-select:none;
}

.thumb img {
    position:absolute;
    top:0;
    left:0;
    width:100%;
    height:100%;
    object-fit:cover;
    pointer-events:none;
}

.thumb.dragging {
    opacity:0.45;
    transform:scale(0.96);
    border:3px solid #0072ff;
    z-index:20;
}

/* 갤러리 삭제 */
.gallery-del-btn {
    position:absolute;
    top:6px;
    right:6px;
    background:rgba(255,255,255,0.9);
    color:#333;
    border-radius:50%;
    width:24px;
    height:24px;
    display:flex;
    justify-content:center;
    align-items:center;
    font-size:12px;
    cursor:pointer;
    border:1px solid #aaa;
    z-index:10;
}

/* 문장 구성 */
.slot-area {
    display:flex;
    flex-wrap:wrap;
    justify-content:flex-start;
    margin-top:12px;
    gap:12px;
}

.slot {
    width:130px;
    height:130px;
    border:2px dashed #cfd9dc;
    border-radius:16px;
    display:flex;
    align-items:center;
    justify-content:center;
    position:relative;
    overflow:hidden;
    background:#f9f9f9;
    transition:all 0.15s;

    cursor:grab;
    touch-action:none;
    user-select:none;
    -webkit-user-select:none;
}

.slot img {
    width:100%;
    height:100%;
    object-fit:cover;
    border-radius:16px;
    pointer-events:none;
}

.slot.dragging {
    opacity:0.45;
    border:3px solid #0072ff;
    transform:scale(0.96);
    z-index:20;
}

.del-btn {
    position:absolute;
    top:6px;
    right:6px;
    background:white;
    border-radius:50%;
    padding:4px 6px;
    font-size:12px;
    cursor:pointer;
    border:1px solid #aaa;
    z-index:10;
}

.mid-buttons {
    display:flex;
    justify-content:center;
    gap:20px;
    margin:12px 0;
}

.bottom-right {
    position:fixed;
    bottom:16px;
    right:16px;
}

@media(max-width:600px) {
    .slot {
        width:95px;
        height:95px;
    }

    .section {
        margin:10px;
        padding:12px;
    }

    .gallery {
        gap:6px;
    }
}
</style>
</head>

<body>

<div class="top">
    <div class="yes-no">
        <button class="btn" id="yesBtn">네</button>
        <button class="btn" id="noBtn">아니요</button>
    </div>
</div>

<div class="section">
    <h3>문장 구성</h3>
    <div class="slot-area" id="slots"></div>
</div>

<div class="mid-buttons">
    <button class="btn" id="clearAll">전체삭제</button>
    <button class="btn" id="readAgain">다시 읽기</button>
</div>

<div class="section">
    <h3>이미지 업로드</h3>
    <input id="imgInput" type="file" accept="image/*" multiple>
    <div class="gallery" id="gallery"></div>
</div>

<div class="bottom-right">
    <button class="btn primary" id="saveHtml">HTML 저장</button>
</div>

<script>

/* =========================
   기본 변수
========================= */

let gallery = [];
let slots = [];

const gEl = document.getElementById("gallery");
const sEl = document.getElementById("slots");
const imgInput = document.getElementById("imgInput");

const yesBtn = document.getElementById("yesBtn");
const noBtn = document.getElementById("noBtn");
const clearAllBtn = document.getElementById("clearAll");
const readAgainBtn = document.getElementById("readAgain");
const saveHtmlBtn = document.getElementById("saveHtml");

let suppressGalleryClickUntil = 0;


/* =========================
   음성 출력
========================= */

const speechQueue = [];
let isSpeaking = false;
const synth = window.speechSynthesis;

function queueAndSpeak(text,onComplete=null) {

    if (!synth || !text || !text.trim()) {
        if(onComplete) onComplete();
        return;
    }

    stopAllSpeech();

    const utterance = new SpeechSynthesisUtterance(text);

    utterance.lang = 'ko-KR';
    utterance.rate = 1.2;

    utterance.onend = () => {
        isSpeaking = false;
        if(onComplete) onComplete();
        processNextSpeech();
    };

    utterance.onerror = () => {
        isSpeaking = false;
        if(onComplete) onComplete();
        processNextSpeech();
    };

    speechQueue.push(utterance);

    if(!isSpeaking) {
        processNextSpeech();
    }
}

function processNextSpeech() {

    if(!isSpeaking && speechQueue.length > 0) {

        isSpeaking = true;

        const next = speechQueue.shift();

        synth.speak(next);
    }
}

function stopAllSpeech() {

    if(synth) synth.cancel();

    speechQueue.length = 0;
    isSpeaking = false;
}


/* =========================
   IndexedDB
   기존 데이터베이스 유지
========================= */

let db;

const DB_NAME = 'aacImageDB';
const STORE_NAME = 'images';
const DB_VERSION = 1;

function openDB() {

    return new Promise((resolve,reject)=>{

        const request = indexedDB.open(DB_NAME,DB_VERSION);

        request.onupgradeneeded = (event)=>{

            db = event.target.result;

            if(!db.objectStoreNames.contains(STORE_NAME)) {

                db.createObjectStore(
                    STORE_NAME,
                    {keyPath:'id',autoIncrement:true}
                );
            }
        };

        request.onsuccess = (event)=>{

            db = event.target.result;

            resolve(db);
        };

        request.onerror = (event)=>{

            reject(event.target.error);
        };
    });
}


/* =========================
   이미지 추가
========================= */

async function addImageToIndexedDB(dataURL,fileName) {

    if(!db) await openDB();

    return new Promise((resolve,reject)=>{

        const transaction = db.transaction(
            [STORE_NAME],
            'readwrite'
        );

        const store = transaction.objectStore(STORE_NAME);

        const request = store.add({
            dataURL,
            fileName
        });

        request.onsuccess = ()=>resolve();

        request.onerror = (event)=>reject(event.target.error);
    });
}


/* =========================
   이미지 순서 저장
========================= */

const GALLERY_ORDER_KEY = 'aacImageDB_galleryOrder_v1';

function saveGalleryOrder() {

    try {

        const order = gallery.map(item=>item.id);

        localStorage.setItem(
            GALLERY_ORDER_KEY,
            JSON.stringify(order)
        );

    } catch(error) {

        console.error("이미지 순서 저장 실패",error);
    }
}

function sortGalleryBySavedOrder(items) {

    let savedOrder = [];

    try {

        savedOrder = JSON.parse(
            localStorage.getItem(GALLERY_ORDER_KEY) || "[]"
        );

        if(!Array.isArray(savedOrder)) {
            savedOrder = [];
        }

    } catch(error) {

        savedOrder = [];
    }

    const orderMap = new Map();

    savedOrder.forEach((id,index)=>{

        orderMap.set(String(id),index);

    });

    return [...items].sort((a,b)=>{

        const ai = orderMap.has(String(a.id))
            ? orderMap.get(String(a.id))
            : Infinity;

        const bi = orderMap.has(String(b.id))
            ? orderMap.get(String(b.id))
            : Infinity;

        return ai-bi;
    });
}


/* =========================
   이미지 불러오기
========================= */

async function loadImagesFromIndexedDB() {

    if(!db) await openDB();

    return new Promise((resolve,reject)=>{

        const transaction = db.transaction(
            [STORE_NAME],
            'readonly'
        );

        const store = transaction.objectStore(STORE_NAME);

        const request = store.getAll();

        request.onsuccess = (event)=>{

            gallery = sortGalleryBySavedOrder(
                event.target.result
            );

            saveGalleryOrder();

            renderGallery();

            resolve(gallery);
        };

        request.onerror = (event)=>{

            reject(event.target.error);
        };
    });
}


/* =========================
   이미지 삭제
========================= */

async function deleteImageFromIndexedDB(id) {

    if(!db) await openDB();

    return new Promise((resolve,reject)=>{

        const transaction = db.transaction(
            [STORE_NAME],
            'readwrite'
        );

        const store = transaction.objectStore(STORE_NAME);

        const request = store.delete(id);

        request.onsuccess = ()=>resolve();

        request.onerror = (event)=>reject(event.target.error);
    });
}

async function deleteGalleryItem(id) {

    if(confirm("이 이미지를 갤러리에서 삭제하시겠습니까?")) {

        await deleteImageFromIndexedDB(id);

        await loadImagesFromIndexedDB();
    }
}


/* =========================
   이미지 업로드 이벤트
========================= */

imgInput.addEventListener("change",async e=>{

    const files = [...e.target.files];

    if(files.length===0) return;

    for(const file of files) {

        try {

            const dataURL = await new Promise((resolve,reject)=>{

                const reader = new FileReader();

                reader.onload = ev=>resolve(ev.target.result);

                reader.onerror = reject;

                reader.readAsDataURL(file);
            });

            await addImageToIndexedDB(
                dataURL,
                file.name.replace(/\.[^/.]+$/,"")
            );

        } catch(error) {

            console.error("이미지 업로드 실패",error);
        }
    }

    e.target.value = '';

    await loadImagesFromIndexedDB();
});


/* =========================
   이미지 배경판 출력
========================= */

function renderGallery() {

    gEl.innerHTML = "";

    gallery.forEach(item=>{

        const d = document.createElement("div");

        d.className = "thumb";

        d.dataset.imageId = String(item.id);

        d.innerHTML = `
            <img src="${item.dataURL}">
            <div class="gallery-del-btn">×</div>
        `;

        /* 이미지 선택 */

        d.addEventListener("click",()=>{

            if(Date.now()<suppressGalleryClickUntil) return;

            slots.push({

                id:createUniqueId(),

                url:item.dataURL,

                name:item.fileName

            });

            renderSlots();

            queueAndSpeak(item.fileName);
        });

        /* 이미지 삭제 */

        d.querySelector(".gallery-del-btn").onclick = e=>{

            e.stopPropagation();

            deleteGalleryItem(item.id);
        };

        gEl.appendChild(d);
    });
}


/* =========================
   문장 구성판 출력
========================= */

function createUniqueId() {

    return Date.now() + "-" +
        Math.random().toString(36).slice(2);
}

function renderSlots() {

    sEl.innerHTML = "";

    slots.forEach((slotData,i)=>{

        const slot = document.createElement("div");

        slot.className = "slot";

        slot.dataset.slotId = String(slotData.id);

        slot.innerHTML = `
            <img src="${slotData.url}">
            <div class="del-btn">×</div>
        `;

        slot.querySelector(".del-btn").onclick = e=>{

            e.stopPropagation();

            slots.splice(i,1);

            renderSlots();
        };

        sEl.appendChild(slot);
    });
}


/* =========================
   공통 드래그 삽입 기능
   기존 이미지는 한 칸씩 밀림
========================= */

function enableDragReorder(container,selector,onReorder) {

    let dragState = null;

    container.addEventListener("pointerdown",e=>{

        if(e.target.closest(".gallery-del-btn,.del-btn")) return;

        const item = e.target.closest(selector);

        if(!item || item.parentElement!==container) return;

        if(e.pointerType==="mouse" && e.button!==0) return;

        dragState = {

            item:item,

            pointerId:e.pointerId,

            startX:e.clientX,

            startY:e.clientY,

            dragging:false
        };
    });

    document.addEventListener("pointermove",e=>{

        if(!dragState) return;

        if(e.pointerId!==dragState.pointerId) return;

        const dx = e.clientX-dragState.startX;

        const dy = e.clientY-dragState.startY;

        /* 작은 움직임은 클릭으로 처리 */

        if(!dragState.dragging && Math.hypot(dx,dy)<8) {
            return;
        }

        dragState.dragging = true;

        dragState.item.classList.add("dragging");

        e.preventDefault();

        const hit = document.elementFromPoint(
            e.clientX,
            e.clientY
        );

        if(!container.contains(hit)) return;

        let target = hit.closest(selector);

        /* 자기 자신 위에서는 이동하지 않음 */

        if(target===dragState.item) return;

        /* 빈 공간에서는 가장 가까운 이미지 찾기 */

        if(!target || target.parentElement!==container) {

            const candidates = [
                ...container.querySelectorAll(selector)
            ].filter(el=>el!==dragState.item);

            if(candidates.length===0) return;

            let nearest = null;

            let minDistance = Infinity;

            candidates.forEach(el=>{

                const rect = el.getBoundingClientRect();

                const cx = rect.left+rect.width/2;

                const cy = rect.top+rect.height/2;

                const distance =
                    Math.pow(e.clientX-cx,2) +
                    Math.pow(e.clientY-cy,2);

                if(distance<minDistance) {

                    minDistance = distance;

                    nearest = el;
                }
            });

            target = nearest;
        }

        if(!target) return;

        const rect = target.getBoundingClientRect();

        const centerX = rect.left+rect.width/2;

        const centerY = rect.top+rect.height/2;

        const sameRow =
            Math.abs(e.clientY-centerY)<rect.height*0.4;

        const insertBefore = sameRow
            ? e.clientX<centerX
            : e.clientY<centerY;

        /* 앞에 삽입 */

        if(insertBefore) {

            if(dragState.item.nextElementSibling!==target) {

                container.insertBefore(
                    dragState.item,
                    target
                );
            }

        }

        /* 뒤에 삽입 */

        else {

            if(target.nextElementSibling!==dragState.item) {

                container.insertBefore(
                    dragState.item,
                    target.nextElementSibling
                );
            }
        }

    },{passive:false});


    /* 드래그 완료 */

    function finishDrag(e) {

        if(!dragState) return;

        if(e && e.pointerId!==dragState.pointerId) return;

        const wasDragging = dragState.dragging;

        dragState.item.classList.remove("dragging");

        if(wasDragging) {

            onReorder();

            if(container===gEl) {

                suppressGalleryClickUntil = Date.now()+500;
            }
        }

        dragState = null;
    }

    document.addEventListener("pointerup",finishDrag);

    document.addEventListener("pointercancel",finishDrag);
}


/* =========================
   배경판 순서 동기화
========================= */

function syncGalleryOrderFromDOM() {

    const itemMap = new Map(

        gallery.map(item=>[
            String(item.id),
            item
        ])

    );

    gallery = [
        ...gEl.querySelectorAll(".thumb")
    ].map(el=>{

        return itemMap.get(el.dataset.imageId);

    }).filter(Boolean);

    /* 자동 저장 */

    saveGalleryOrder();
}


/* =========================
   문장 구성 순서 동기화
========================= */

function syncSlotOrderFromDOM() {

    const itemMap = new Map(

        slots.map(item=>[
            String(item.id),
            item
        ])

    );

    slots = [
        ...sEl.querySelectorAll(".slot")
    ].map(el=>{

        return itemMap.get(el.dataset.slotId);

    }).filter(Boolean);

    renderSlots();
}


/* =========================
   드래그 기능 활성화
========================= */

/* 이미지 배경판 */

enableDragReorder(
    gEl,
    ".thumb",
    syncGalleryOrderFromDOM
);

/* 문장 구성판 */

enableDragReorder(
    sEl,
    ".slot",
    syncSlotOrderFromDOM
);


/* =========================
   네 / 아니요
========================= */

yesBtn.addEventListener("click",()=>{

    queueAndSpeak("네");

});

noBtn.addEventListener("click",()=>{

    queueAndSpeak("아니요");

});


/* =========================
   전체 삭제
========================= */

clearAllBtn.addEventListener("click",()=>{

    slots.length = 0;

    renderSlots();

    stopAllSpeech();
});


/* =========================
   다시 읽기
========================= */

readAgainBtn.addEventListener("click",()=>{

    stopAllSpeech();

    if(slots.length===0) {

        queueAndSpeak("선택된 이미지가 없어요.");

        return;
    }

    const allTexts = slots
        .map(slot=>slot.name)
        .join(" ");

    queueAndSpeak(allTexts);
});


/* =========================
   HTML 저장
========================= */

saveHtmlBtn.addEventListener("click",()=>{

    const htmlContent =
        document.documentElement.outerHTML;

    const blob = new Blob(
        [htmlContent],
        {type:"text/html"}
    );

    const url = URL.createObjectURL(blob);

    const a = document.createElement("a");

    a.href = url;

    a.download = "aac_tool.html";

    document.body.appendChild(a);

    a.click();

    document.body.removeChild(a);

    URL.revokeObjectURL(url);

    queueAndSpeak("HTML을 저장했어요!");
});


/* =========================
   시작
========================= */

document.addEventListener("DOMContentLoaded",async()=>{

    try {

        await openDB();

        await loadImagesFromIndexedDB();

    } catch(error) {

        console.error("초기화 실패",error);

        alert("이미지 데이터베이스를 불러오지 못했습니다.");
    }
});

</script>

</body>
</html>
