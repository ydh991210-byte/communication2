<!doctype html>
<html lang="ko">
<head>
<meta charset="utf-8">
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
    transition:all 0.2s;
    box-shadow:0 2px 4px rgba(0,0,0,0.1);
}

.btn:hover {
    transform:translateY(-2px);
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
    transition:transform 0.2s;
    border:2px solid #e0e0e0;

    touch-action:none;
    user-select:none;
    -webkit-user-select:none;
    -webkit-user-drag:none;
}

.thumb:hover {
    transform:scale(1.05);
    border-color:#0072ff;
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
   갤러리 삭제 버튼
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

    touch-action:manipulation;
}

.gallery-del-btn:hover {
    background:rgba(255,0,0,0.7);
    color:white;
    border-color:red;
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

    transition:all 0.2s;

    cursor:grab;

    touch-action:none;
    user-select:none;
    -webkit-user-select:none;
    -webkit-user-drag:none;
}

.slot:hover {
    background:#eef6ff;
    border-color:#0072ff;
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
    box-shadow:0 1px 3px rgba(0,0,0,0.2);

    z-index:10;

    touch-action:manipulation;
}


/* =========================
   스마트폰 앱 아이콘 흔들림
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
   손가락을 따라다니는 이미지
========================= */

.drag-ghost {

    position:fixed !important;

    z-index:99999 !important;

    pointer-events:none !important;

    opacity:0.96;

    transform:scale(1.08);

    box-shadow:0 8px 25px rgba(0,0,0,0.3);

    border:3px solid #0072ff !important;

    animation:none !important;

}


/* 원래 이미지가 있던 자리 */

.drag-placeholder {

    visibility:hidden !important;

    pointer-events:none !important;

    animation:none !important;

}


/* =========================
   버튼
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

    .section {
        margin:10px;
        padding:12px;
    }

    .gallery {
        gap:6px;
    }

    .slot {
        width:95px;
        height:95px;
    }

}

</style>
</head>


<body>


<!-- =========================
     상단 버튼
========================= -->

<div class="top">

    <div class="yes-no">

        <button class="btn" id="yesBtn">
            네
        </button>

        <button class="btn" id="noBtn">
            아니요
        </button>

    </div>

</div>


<!-- =========================
     문장 구성판
========================= -->

<div class="section">

    <h3>문장 구성</h3>

    <div class="slot-area" id="slots"></div>

</div>


<!-- =========================
     중간 버튼
========================= -->

<div class="mid-buttons">

    <button class="btn" id="clearAll">
        전체삭제
    </button>

    <button class="btn" id="readAgain">
        다시 읽기
    </button>

</div>


<!-- =========================
     이미지 배경판
========================= -->

<div class="section">

    <h3>이미지 업로드</h3>

    <input
        id="imgInput"
        type="file"
        accept="image/*"
        multiple
    >

    <div class="gallery" id="gallery"></div>

</div>


<!-- =========================
     HTML 저장
========================= -->

<div class="bottom-right">

    <button class="btn primary" id="saveHtml">
        HTML 저장
    </button>

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

if(!synth) {

    alert(
        "사용하시는 브라우저는 음성 합성을 지원하지 않습니다."
    );

}


function queueAndSpeak(text,onComplete=null) {

    if(!synth || !text || text.trim()==="") {

        if(onComplete) onComplete();

        return;

    }

    stopAllSpeech();

    const utterance = new SpeechSynthesisUtterance(text);

    utterance.lang = "ko-KR";

    utterance.rate = 1.2;

    utterance.onend = ()=>{

        isSpeaking = false;

        if(onComplete) onComplete();

        processNextSpeech();

    };

    utterance.onerror = ()=>{

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

    if(!isSpeaking && speechQueue.length>0) {

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

const DB_NAME = "aacImageDB";

const STORE_NAME = "images";

const DB_VERSION = 1;


function openDB() {

    return new Promise((resolve,reject)=>{

        const request = indexedDB.open(
            DB_NAME,
            DB_VERSION
        );

        request.onupgradeneeded = (event)=>{

            db = event.target.result;

            if(!db.objectStoreNames.contains(STORE_NAME)) {

                db.createObjectStore(
                    STORE_NAME,
                    {
                        keyPath:"id",
                        autoIncrement:true
                    }
                );

            }

        };

        request.onsuccess = (event)=>{

            db = event.target.result;

            console.log("IndexedDB 연결 성공");

            resolve(db);

        };

        request.onerror = (event)=>{

            console.error(
                "IndexedDB 오류",
                event.target.error
            );

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
            "readwrite"
        );

        const store = transaction.objectStore(STORE_NAME);

        const request = store.add({

            dataURL:dataURL,

            fileName:fileName

        });

        request.onsuccess = ()=>resolve();

        request.onerror = (event)=>{

            reject(event.target.error);

        };

    });

}


/* ==================================
   이미지 순서 저장
================================== */

const GALLERY_ORDER_KEY =
    "aacImageDB_galleryOrder_v1";


function saveGalleryOrder() {

    try {

        const order = gallery.map(item=>item.id);

        localStorage.setItem(
            GALLERY_ORDER_KEY,
            JSON.stringify(order)
        );

    } catch(error) {

        console.error("순서 저장 오류",error);

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
            : Number.MAX_SAFE_INTEGER;

        const bi = orderMap.has(String(b.id))
            ? orderMap.get(String(b.id))
            : Number.MAX_SAFE_INTEGER;

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
            "readonly"
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
            "readwrite"
        );

        const store = transaction.objectStore(STORE_NAME);

        const request = store.delete(id);

        request.onsuccess = ()=>resolve();

        request.onerror = (event)=>{

            reject(event.target.error);

        };

    });

}


async function deleteGalleryItem(id) {

    if(confirm("이 이미지를 삭제하시겠습니까?")) {

        await deleteImageFromIndexedDB(id);

        await loadImagesFromIndexedDB();

    }

}


/* ==================================
   이미지 업로드
================================== */

imgInput.addEventListener("change",async e=>{

    const files = [...e.target.files];

    e.target.value = "";

    if(files.length===0) return;

    for(const file of files) {

        try {

            const dataURL = await new Promise((resolve,reject)=>{

                const reader = new FileReader();

                reader.onload = ev=>{

                    resolve(ev.target.result);

                };

                reader.onerror = reject;

                reader.readAsDataURL(file);

            });

            await addImageToIndexedDB(
                dataURL,
                file.name.replace(/\.[^/.]+$/,"")
            );

        } catch(error) {

            console.error(
                "이미지 업로드 오류",
                error
            );

        }

    }

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

            if(Date.now()<suppressGalleryClickUntil) {

                return;

            }

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


        /* 이미지 삭제 */

        slot.querySelector(".del-btn").onclick = e=>{

            e.stopPropagation();

            slots.splice(i,1);

            renderSlots();

        };


        sEl.appendChild(slot);

    });

}


/* ==================================
   스마트폰 앱 아이콘 방식 드래그
================================== */

function enableDragReorder(
    container,
    selector,
    onReorder
) {

    const HOLD_DELAY = 700;

    let dragState = null;


    /* =========================
       터치 또는 마우스 누르기
    ========================= */

    container.addEventListener("pointerdown",e=>{

        if(e.target.closest(".gallery-del-btn,.del-btn")) {

            return;

        }

        const item = e.target.closest(selector);

        if(!item || item.parentElement!==container) {

            return;

        }

        if(e.pointerType==="mouse" && e.button!==0) {

            return;

        }

        if(dragState) return;


        dragState = {

            item:item,

            pointerId:e.pointerId,

            startX:e.clientX,

            startY:e.clientY,

            ready:false,

            dragging:false,

            cancelled:false,

            timer:null,

            ghost:null,

            placeholder:null,

            originalNextSibling:item.nextElementSibling

        };


        const state = dragState;


        /* 0.7초 길게 누르기 */

        state.timer = setTimeout(()=>{

            if(dragState!==state) return;

            state.ready = true;

            /* 편집 모드 시작 */

            container.classList.add("edit-mode");

            state.item.classList.add("long-pressed");

        },HOLD_DELAY);

    });


    /* =========================
       손가락 이동
    ========================= */

    document.addEventListener("pointermove",e=>{

        if(!dragState) return;

        const state = dragState;

        if(e.pointerId!==state.pointerId) return;


        const dx = e.clientX-state.startX;

        const dy = e.clientY-state.startY;

        const distance = Math.hypot(dx,dy);


        /* 길게 누르기 전 이동하면 취소 */

        if(!state.ready) {

            if(distance>14) {

                clearTimeout(state.timer);

                state.cancelled = true;

            }

            return;

        }


        /* 실제 드래그 시작 */

        if(!state.dragging && distance<8) {

            return;

        }


        /* =========================
           드래그 최초 시작
        ========================= */

        if(!state.dragging) {

            state.dragging = true;

            const rect = state.item.getBoundingClientRect();


            /* 원래 자리 유지용 빈 공간 */

            const placeholder = state.item.cloneNode(false);

            placeholder.className =
                state.item.classList.contains("thumb")
                ? "thumb drag-placeholder"
                : "slot drag-placeholder";

            placeholder.removeAttribute("id");

            state.item.replaceWith(placeholder);

            state.placeholder = placeholder;


            /* 손가락을 따라다니는 이미지 */

            const ghost = state.item.cloneNode(true);

            ghost.classList.remove(
                "long-pressed",
                "dragging"
            );

            ghost.classList.add("drag-ghost");


            ghost.style.width = rect.width+"px";

            ghost.style.height = rect.height+"px";

            ghost.style.left = rect.left+"px";

            ghost.style.top = rect.top+"px";

            ghost.style.padding = "0";

            ghost.style.margin = "0";

            ghost.style.boxSizing = "border-box";

            ghost.style.animation = "none";


            document.body.appendChild(ghost);


            state.ghost = ghost;


            /* 손가락과 이미지 사이 간격 */

            state.offsetX = state.startX-rect.left;

            state.offsetY = state.startY-rect.top;

        }


        e.preventDefault();


        /* =========================
           이미지가 손가락을 따라 이동
        ========================= */

        state.ghost.style.left =
            (e.clientX-state.offsetX)+"px";

        state.ghost.style.top =
            (e.clientY-state.offsetY)+"px";


        /* =========================
           손가락 아래 이미지 확인
        ========================= */

        const hit = document.elementFromPoint(
            e.clientX,
            e.clientY
        );

        let target = hit ? hit.closest(selector) : null;


        if(
            !target ||
            target.parentElement!==container ||
            target===state.placeholder
        ) {

            /* 빈 공간에서는 가장 가까운 이미지 검색 */

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

                    Math.pow(e.clientX-cx,2) +

                    Math.pow(e.clientY-cy,2);


                if(d<minDistance) {

                    minDistance = d;

                    nearest = el;

                }

            });


            target = nearest;

        }


        if(!target) return;


        /* =========================
           삽입 방향 계산
        ========================= */

        const rect = target.getBoundingClientRect();

        const centerX = rect.left+rect.width/2;

        const centerY = rect.top+rect.height/2;


        const sameRow =

            Math.abs(e.clientY-centerY)<rect.height*0.45;


        const insertBefore = sameRow

            ? e.clientX<centerX

            : e.clientY<centerY;


        /* =========================
           기존 이미지를 한 칸씩 밀기
        ========================= */

        if(insertBefore) {

            if(
                state.placeholder.nextElementSibling!==target
            ) {

                container.insertBefore(
                    state.placeholder,
                    target
                );

            }

        } else {

            if(
                target.nextElementSibling!==state.placeholder
            ) {

                container.insertBefore(
                    state.placeholder,
                    target.nextElementSibling
                );

            }

        }

    },{passive:false});


    /* =========================
       드래그 종료
    ========================= */

    function finishDrag(e,cancelled=false) {

        if(!dragState) return;

        const state = dragState;

        if(e && e.pointerId!==state.pointerId) {

            return;

        }

        clearTimeout(state.timer);


        /* 이동 취소 시 원래 자리로 복귀 */

        if(cancelled && state.dragging) {

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

        if(state.dragging && state.placeholder) {

            state.placeholder.replaceWith(state.item);

        }


        /* 손가락을 따라다니는 이미지 제거 */

        if(state.ghost) {

            state.ghost.remove();

        }


        /* 흔들림 제거 */

        state.item.classList.remove(
            "long-pressed",
            "dragging"
        );


        /* 편집 모드 종료 */

        container.classList.remove("edit-mode");


        /* 실제 이동이 완료된 경우 */

        if(state.dragging && !cancelled) {

            onReorder();

        }


        /* 이미지 선택 방지 */

        if(
            container===gEl &&
            (
                state.ready ||
                state.dragging ||
                state.cancelled ||
                cancelled
            )
        ) {

            suppressGalleryClickUntil =
                Date.now()+700;

        }


        dragState = null;

    }


    /* 손가락 놓기 */

    document.addEventListener("pointerup",e=>{

        finishDrag(e,false);

    });


    /* 이동 취소 */

    document.addEventListener("pointercancel",e=>{

        finishDrag(e,true);

    });


    /* 모바일 기본 메뉴 방지 */

    container.addEventListener("contextmenu",e=>{

        if(e.target.closest(selector)) {

            e.preventDefault();

        }

    });

}


/* ==================================
   배경판 순서 저장
================================== */

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


/* ==================================
   문장 구성판 순서 저장
================================== */

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


/* ==================================
   드래그 기능 활성화
================================== */


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

    slots.length = 0;

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

        console.error("초기화 오류",error);

        alert(
            "이미지 데이터베이스를 불러오지 못했습니다."
        );

    }

});

</script>

</body>
</html>
