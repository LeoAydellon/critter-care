# Tend LIVE — Cloud integration blueprint (code ready to drop in)
Built by Athena 2026-07-06 night. Morning = paste Firebase keys (Step 5 of FIREBASE-SETUP-morning.md) → assemble into index.html → deploy → test on Leo + Raphael's phones.

## Architecture (model A — each household = its own farm)
- **Firestore doc:** `farms/{farmId}` = `{ modules: [...], state: {...}, asks: {...}, updatedAt }`.
- **Photos → Firebase Storage** (NOT base64 in the doc; Firestore has a 1MB doc cap). On add-photo: upload the compressed JPEG to `farms/{farmId}/{uuid}.jpg`, store the returned **download URL** string in place of the base64. Renderers already just use the string as an `<img src>`, so URLs work unchanged.
- **farmId** comes from the URL `?farm=xxxx`. No farm param → generate a new id, seed defaults, push it into the URL. **Share** = share the current URL.
- **Live sync:** one `onSnapshot(farmDoc)` listener → on any change, set MODULES/state/asks from cloud and re-render. Every phone updates instantly.
- Keep IndexedDB as an **offline cache/fallback** (write-through), so it still shows last-known data with no signal.

## Code blocks (ready)

### 1. Head — load Firebase SDK + init (replace nothing; add near top of <script>, make script `type="module"`)
```html
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
import { getFirestore, doc, getDoc, setDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";
import { getStorage, ref, uploadString, getDownloadURL } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-storage.js";

// ▼▼▼ PASTE LEO'S firebaseConfig BLOCK HERE (from setup Step 2) ▼▼▼
const firebaseConfig = { /* apiKey, authDomain, projectId, storageBucket, ... */ };
// ▲▲▲

const fbApp = initializeApp(firebaseConfig);
const db = getFirestore(fbApp);
const storage = getStorage(fbApp);
window.__tendCloud = { db, storage, doc, getDoc, setDoc, onSnapshot, ref, uploadString, getDownloadURL };
</script>
```

### 2. farmId from URL (or create one)
```js
function getFarmId(){
  var u=new URL(location.href); var f=u.searchParams.get('farm');
  if(!f){ f='farm-'+Math.random().toString(36).slice(2,8); u.searchParams.set('farm',f); history.replaceState(null,'',u); }
  return f;
}
var FARM_ID = getFarmId();
function farmRef(){ var c=window.__tendCloud; return c.doc(c.db,'farms',FARM_ID); }
```

### 3. Cloud load + realtime (replace the idbGet loads in boot())
```js
async function cloudBoot(){
  var c=window.__tendCloud;
  var snap = await c.getDoc(farmRef());
  if(snap.exists()){ var d=snap.data(); MODULES=d.modules||[]; state=d.state||{}; asks=d.asks||{}; }
  else { MODULES=loadModulesFromLS(); state={}; asks={}; await saveCloud(); }   // seed a new farm
  renderToday();
  // live updates from any phone:
  c.onSnapshot(farmRef(), function(s){ if(!s.exists())return; var d=s.data(); MODULES=d.modules||[]; state=d.state||{}; asks=d.asks||{}; renderToday(); if(document.getElementById('view-setup').offsetParent) renderSetup(); });
}
```

### 4. Save to cloud (replace saveModules/saveState/saveAsks bodies)
```js
async function saveCloud(){
  var c=window.__tendCloud;
  try{ await c.setDoc(farmRef(), {modules:MODULES,state:state,asks:asks,updatedAt:Date.now()}); }catch(e){ console.warn('cloud save',e); }
}
function saveModules(){ saveCloud(); try{localStorage.setItem(LS_KEY,JSON.stringify(MODULES));}catch(e){} }
function saveState(){ saveCloud(); }
function saveAsks(){ saveCloud(); }
```

### 5. Photo → Storage (the one real change to photo handling)
Wherever a photo data-URL is currently stored into MODULES/state, first upload it and store the URL:
```js
async function uploadPhoto(dataUrl){
  var c=window.__tendCloud;
  var key='farms/'+FARM_ID+'/'+Date.now()+'-'+Math.random().toString(36).slice(2,7)+'.jpg';
  var r=c.ref(c.storage,key);
  await c.uploadString(r, dataUrl, 'data_url');
  return await c.getDownloadURL(r);   // <-- store THIS string instead of the base64
}
// e.g. in resizeImage callback: uploadPhoto(d).then(function(url){ state[id]=Object.assign({},state[id],{photo:url}); saveState(); renderToday(); });
```

## Assembly steps (morning, ~15 min with Leo)
1. Copy `index.html` → keep as `index-local-backup.html` (safety).
2. Make the main `<script>` `type="module"`; paste block **1** at top with Leo's keys.
3. Add blocks **2–5**; replace the three `save*` functions and the `boot()` load lines; route the two photo-save spots through `uploadPhoto`.
4. Deploy; open `?farm=ravens-roost` on Leo's phone, build the real setup once (it saves to cloud).
5. Open the SAME link on Raphael's phone → confirm it shows Leo's setup + photos, live. Mark a task done on one phone → watch it update on the other.
6. Ship. Share the farm link with the kids.

## Notes / honest flags
- Rules are MVP-permissive (farm ID = the access key). Fine to launch; can add real auth later if it goes big.
- Free tier limits (Firestore 50k reads/20k writes/day, Storage 5GB/1GB-day download) are FAR above a family chore app. No cost.
- Existing local data on Leo's phone doesn't auto-move to the cloud — the first cloud farm starts fresh, so Leo rebuilds (or we one-time import his backup file into the new farm). **Decide in the morning: rebuild vs import his backup.**
