<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">

<title>AAC 음성출력 도구</title>

<style>

/* =========================
   기본 디자인
========================= */

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


/* =========================
   이미지 배경판
========================= */

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
    cursor:pointer;
    transition:transform 0.15s;
    border:2px solid #e0e0e0;

    /* 스크롤 허용 */
    touch-action:auto;

    user-select:none;
    -webkit-user-select:none;
    -webkit-touch-callout:none;
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


/* =========================
   배경판 삭제 버튼
========================= */

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


/* =========================
   문장 구성판
========================= */

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

    /* 스크롤 허용 */
    touch-action:auto;

    user-select:none;
    -webkit-user-select:none;
    -webkit-touch-callout:none;
}

.slot img {
    width:100%;
    height:100%;
    object-fit:cover;
    border-radius:16px;
    pointer-events:none;
}


/* =========================
   문장 삭제 버튼
========================= */

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


/* =========================
   앱 아이콘 흔들림 효과
========================= */

@keyframes appWiggle {

    0% {
        transform:rotate(-1.3deg);
    }

    50% {
        transform:rotate(1.3deg);
    }

    100% {
        transform:rotate(-1.3deg);
    }

}


/* 편집 모드 */

.gallery.edit-mode .thumb:not(.drag-placeholder),

.slot-area.edit-mode .slot:not(.drag-placeholder) {

    animation:appWiggle 0.18s infinite alternate;

}


/* 선택된 이미지 */

.thumb.long-pressed,
.slot.long-pressed {

    outline:3px solid #ff9800;
    outline-offset:2px;

}


/* =========================
   손가락을 따라가는 이미지
========================= */

.drag-ghost {

    position:fixed !important;

    z-index:99999 !important;

    pointer-events:none !important;

    opacity:0.96;

    transform:scale(1.08);

    transform-origin:center;

    box-shadow:0 8px 25px rgba(0,0,0,0.3);

    border:3px solid #0072ff !important;

    animation:none !important;

}


/* 원래 이미지 자리 */

.drag-placeholder {

    visibility:hidden !important;

    pointer-events:none !important;

    animation:none !important;

}


/* =========================
   중간 버튼
========================= */

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


/* =========================
   모바일
========================= */

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


<!-- 상단 버튼 -->

<div class="top">

    <div class="yes-no">

        <button class="btn" id="yesBtn">네</button>

        <button class="btn" id="noBtn">아니요</button>

    </div>

</div>


<!-- 문장 구성판 -->

<div class="section">

    <h3>문장 구성</h3>

    <div class="slot-area" id="slots"></div>

</div>


<!-- 중간 버튼 -->

<div class="mid-buttons">

    <button class="btn" id="clearAll">전체삭제</button>

    <button class="btn" id="readAgain">다시 읽기</button>

</div>


<!-- 이미지 배경판 -->

<div class="section">

    <h3>이미지 업로드</h3>

    <input id="imgInput" type="file" accept="image/*" multiple>

    <div class="gallery" id="gallery"></div>

</div>


<!-- HTML 저장 -->

<div class="bottom-right">

    <button class="btn primary" id="saveHtml">HTML 저장</button>

</div>


<script>

/* ==================================
   기본 변수
================================== */

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

let lastTouchAt = 0;


function createUniqueId() {

    return Date.now() + "-" +
        Math.random().toString(36).slice(2);

}


/* ==================================
   음성 출력
================================== */

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


/* ==================================
   IndexedDB
   기존 데이터베이스 유지
================================== */

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


/* ==================================
   이미지 추가
================================== */

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


/* ==================================
   이미지 순서 저장
================================== */

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


/* ==================================
   이미지 불러오기
================================== */

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


/* ==================================
   이미지 삭제
================================== */

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


/* ==================================
   이미지 업로드
================================== */

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


/* ==================================
   이미지 배경판 출력
================================== */

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


/* ==================================
   문장 구성판 출력
================================== */

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


        /* 삭제 */

        slot.querySelector(".del-btn").onclick = e=>{

            e.stopPropagation();

            slots.splice(i,1);

            renderSlots();

        };


        sEl.appendChild(slot);

    });

}


/* ==========================================
   공통 드래그 보조 함수
========================================== */

function createDragState(item,x,y,id,type) {

    const state = {

        item,

        startX:x,

        startY:y,

        id,

        type,

        ready:false,

        dragging:false,

        cancelled:false,

        timer:null,

        ghost:null,

        placeholder:null,

        originalNextSibling:item.nextElementSibling,

        offsetX:0,

        offsetY:0

    };


    state.timer = setTimeout(()=>{

        state.ready = true;

        item.classList.add("long-pressed");

        item.parentElement.classList.add("edit-mode");

    },700);


    return state;

}


/* ==========================================
   드래그 시작
========================================== */

function beginVisualDrag(state) {

    if(state.dragging) return;

    state.dragging = true;

    const rect = state.item.getBoundingClientRect();


    /* 원래 위치 표시 */

    const placeholder = state.item.cloneNode(false);

    placeholder.className =

        state.item.classList.contains("thumb")

        ? "thumb drag-placeholder"

        : "slot drag-placeholder";


    state.item.replaceWith(placeholder);

    state.placeholder = placeholder;


    /* 손가락을 따라다니는 이미지 */

    const ghost = state.item.cloneNode(true);

    ghost.classList.remove("long-pressed");

    ghost.classList.add("drag-ghost");


    ghost.style.width = rect.width+"px";

    ghost.style.height = rect.height+"px";

    ghost.style.left = rect.left+"px";

    ghost.style.top = rect.top+"px";

    ghost.style.padding = "0";

    ghost.style.margin = "0";

    ghost.style.boxSizing = "border-box";


    document.body.appendChild(ghost);

    state.ghost = ghost;


    state.offsetX = state.startX-rect.left;

    state.offsetY = state.startY-rect.top;

}


/* ==========================================
   드래그 중 이미지 위치 변경
========================================== */

function moveVisualDrag(state,x,y) {

    if(!state.ready || state.cancelled) return;


    const distance = Math.hypot(

        x-state.startX,

        y-state.startY

    );


    if(!state.dragging && distance<8) return;


    if(!state.dragging) {

        beginVisualDrag(state);

    }


    /* 손가락 따라가기 */

    state.ghost.style.left =

        (x-state.offsetX)+"px";

    state.ghost.style.top =

        (y-state.offsetY)+"px";


    const container = state.item.parentElement ||

        state.placeholder.parentElement;


    if(!container) return;


    /* 손가락 아래 대상 찾기 */

    const hit = document.elementFromPoint(x,y);

    let target = hit ? hit.closest(
        container===gEl ? ".thumb" : ".slot"
    ) : null;


    if(

        !target ||

        target.parentElement!==container ||

        target===state.placeholder

    ) {

        const selector = container===gEl

            ? ".thumb"

            : ".slot";


        const candidates = [

            ...container.querySelectorAll(selector)

        ].filter(el=>el!==state.placeholder);


        if(candidates.length===0) return;


        let nearest = null;

        let minDistance = Infinity;


        candidates.forEach(el=>{

            const r = el.getBoundingClientRect();

            const cx = r.left+r.width/2;

            const cy = r.top+r.height/2;


            const d =

                Math.pow(x-cx,2)+

                Math.pow(y-cy,2);


            if(d<minDistance) {

                minDistance = d;

                nearest = el;

            }

        });


        target = nearest;

    }


    if(!target) return;


    /* 삽입 위치 판단 */

    const rect = target.getBoundingClientRect();

    const centerX = rect.left+rect.width/2;

    const centerY = rect.top+rect.height/2;


    const sameRow =

        Math.abs(y-centerY)<rect.height*0.45;


    const insertBefore = sameRow

        ? x<centerX

        : y<centerY;


    /* 한 칸씩 밀기 */

    if(insertBefore) {

        if(state.placeholder.nextElementSibling!==target) {

            container.insertBefore(

                state.placeholder,

                target

            );

        }

    } else {

        if(target.nextElementSibling!==state.placeholder) {

            container.insertBefore(

                state.placeholder,

                target.nextElementSibling

            );

        }

    }

}


/* ==========================================
   드래그 종료 및 정리
========================================== */

function finishVisualDrag(state,commit) {

    if(!state) return;


    clearTimeout(state.timer);


    const container = state.item.parentElement ||

        state.placeholder?.parentElement;


    if(state.dragging && state.placeholder) {

        /* 취소 시 원래 위치로 복귀 */

        if(!commit) {

            if(

                state.originalNextSibling &&

                state.originalNextSibling.parentNode===container

            ) {

                container.insertBefore(

                    state.placeholder,

                    state.originalNextSibling

                );

            } else {

                container.appendChild(state.placeholder);

            }

        }


        /* 원래 이미지 복구 */

        state.placeholder.replaceWith(state.item);

    }


    if(state.ghost) {

        state.ghost.remove();

    }


    state.item.classList.remove(

        "long-pressed",

        "dragging"

    );


    if(container) {

        container.classList.remove("edit-mode");

    }


    /* 순서 저장 */

    if(commit && state.dragging) {

        if(container===gEl) {

            syncGalleryOrderFromDOM();

        } else if(container===sEl) {

            syncSlotOrderFromDOM();

        }

    }


    /* 터치 후 이미지 선택 방지 */

    if(

        container===gEl &&

        (

            state.ready ||

            state.dragging ||

            state.cancelled

        )

    ) {

        suppressGalleryClickUntil=Date.now()+800;

    }

}


/* ==========================================
   모바일 터치 드래그
   일반 스크롤은 허용
========================================== */

function enableTouchDrag(container,selector) {

    let state = null;


    /* 터치 시작 */

    container.addEventListener("touchstart",e=>{

        if(e.touches.length!==1) return;

        if(e.target.closest(".gallery-del-btn,.del-btn")) {

            return;

        }


        const item = e.target.closest(selector);

        if(!item || item.parentElement!==container) return;

        if(state) return;


        const touch = e.changedTouches[0];


        state = createDragState(

            item,

            touch.clientX,

            touch.clientY,

            touch.identifier,

            "touch"

        );

    },{passive:true});


    /* 손가락 이동 */

    document.addEventListener("touchmove",e=>{

        if(!state) return;


        if(e.touches.length>1) {

            finishVisualDrag(state,false);

            state=null;

            return;

        }


        const touch = [...e.changedTouches].find(

            t=>t.identifier===state.id

        );


        if(!touch) return;


        const distance = Math.hypot(

            touch.clientX-state.startX,

            touch.clientY-state.startY

        );


        /* 길게 누르기 전 움직이면 스크롤 */

        if(!state.ready) {

            if(distance>12) {

                clearTimeout(state.timer);

                state.cancelled=true;

            }

            return;

        }


        /* 길게 누른 뒤에는 드래그 */

        e.preventDefault();


        moveVisualDrag(

            state,

            touch.clientX,

            touch.clientY

        );

    },{passive:false});


    /* 손가락 놓기 */

    document.addEventListener("touchend",e=>{

        if(!state) return;


        const touch = [...e.changedTouches].find(

            t=>t.identifier===state.id

        );


        if(!touch) return;


        finishVisualDrag(state,true);

        state=null;

        lastTouchAt=Date.now();

    },{passive:true});


    /* 터치 취소 */

    document.addEventListener("touchcancel",e=>{

        if(!state) return;


        const touch = [...e.changedTouches].find(

            t=>t.identifier===state.id

        );


        if(!touch) return;


        finishVisualDrag(state,false);

        state=null;

        lastTouchAt=Date.now();

    },{passive:true});


    /* 길게 누르기 기본 메뉴 방지 */

    container.addEventListener("contextmenu",e=>{

        if(e.target.closest(selector)) {

            e.preventDefault();

        }

    });

}


/* ==========================================
   PC 마우스 드래그
========================================== */

function enableMouseDrag(container,selector) {

    let state = null;


    /* 마우스 누르기 */

    container.addEventListener("mousedown",e=>{

        /* 모바일의 가상 마우스 이벤트 무시 */

        if(Date.now()-lastTouchAt<800) return;

        if(e.button!==0) return;


        if(e.target.closest(".gallery-del-btn,.del-btn")) {

            return;

        }


        const item = e.target.closest(selector);

        if(!item || item.parentElement!==container) return;

        if(state) return;


        state = createDragState(

            item,

            e.clientX,

            e.clientY,

            null,

            "mouse"

        );

    });


    /* 마우스 이동 */

    document.addEventListener("mousemove",e=>{

        if(!state) return;


        const distance = Math.hypot(

            e.clientX-state.startX,

            e.clientY-state.startY

        );


        if(!state.ready) {

            if(distance>12) {

                clearTimeout(state.timer);

                state.cancelled=true;

            }

            return;

        }


        e.preventDefault();


        moveVisualDrag(

            state,

            e.clientX,

            e.clientY

        );

    });


    /* 마우스 놓기 */

    document.addEventListener("mouseup",()=>{

        if(!state) return;


        finishVisualDrag(state,true);

        state=null;

    });

}


/* ==========================================
   배경판 순서 동기화
========================================== */

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


    saveGalleryOrder();

}


/* ==========================================
   문장 구성판 순서 동기화
========================================== */

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


/* ==========================================
   드래그 기능 활성화
========================================== */


/* 이미지 배경판 */

enableTouchDrag(

    gEl,

    ".thumb"

);

enableMouseDrag(

    gEl,

    ".thumb"

);


/* 문장 구성판 */

enableTouchDrag(

    sEl,

    ".slot"

);

enableMouseDrag(

    sEl,

    ".slot"

);


/* ==================================
   네 / 아니요
================================== */

yesBtn.addEventListener("click",()=>{

    queueAndSpeak("네");

});


noBtn.addEventListener("click",()=>{

    queueAndSpeak("아니요");

});


/* ==================================
   전체 삭제
================================== */

clearAllBtn.addEventListener("click",()=>{

    slots.length=0;

    renderSlots();

    stopAllSpeech();

});


/* ==================================
   다시 읽기
================================== */

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


/* ==================================
   HTML 저장
================================== */

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


    setTimeout(()=>{

        URL.revokeObjectURL(url);

    },1000);


    queueAndSpeak("HTML을 저장했어요!");

});


/* ==================================
   프로그램 시작
================================== */

document.addEventListener("DOMContentLoaded",async()=>{

    try {

        await openDB();

        await loadImagesFromIndexedDB();

        console.log("AAC 프로그램 준비 완료");

    } catch(error) {

        console.error("초기화 실패",error);

        alert("이미지 데이터베이스를 불러오지 못했습니다.");

    }

});

</script>

</body>
</html>
