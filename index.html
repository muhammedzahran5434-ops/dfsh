/* WEBTREX cloud sync: drop-in upgrade, no other edits to index.html needed.
   Put <script src="webtrex-cloud.js"></script> in <head>, right AFTER the firebase-*-compat.js scripts. */
(function () {
  'use strict';
  if (typeof firebase === 'undefined') return;

  var CFG = { // same project as the app (keep in sync if you ever change it)
    apiKey: 'AIzaSyDltBbZJWcVSywBpxWEwhoGxenePhwrO-0',
    authDomain: 'webtrex-portfolio.firebaseapp.com',
    projectId: 'webtrex-portfolio',
    storageBucket: 'webtrex-portfolio.firebasestorage.app',
    messagingSenderId: '536269316009',
    appId: '1:536269316009:web:b90d54772768e0ab9caeb8',
    measurementId: 'G-S1N1E22ZTX'
  };
  var ADMIN = 'webtrex@gmail.com';
  var K = { p: 'webtrex_cms_projects_v1', m: 'webtrex_cms_messages_v1', s: 'webtrex_cms_settings_v1' };

  if (!firebase.apps.length) firebase.initializeApp(CFG);
  var db = firebase.firestore();
  var auth = firebase.auth();
  var storage = typeof firebase.storage === 'function' ? firebase.storage() : null;

  var rawSet = Storage.prototype.setItem;
  var mute = false, subs = [], sent = {}, snap = { p: [], m: [] }, pill;

  function admin() { return !!auth.currentUser && (auth.currentUser.email || '').toLowerCase() === ADMIN; }
  function clean(x) { return JSON.parse(JSON.stringify(x)); }
  function sj(v) { // key-order-stable JSON for diffing
    return JSON.stringify(v, function (k, val) {
      if (val && typeof val === 'object' && !Array.isArray(val)) {
        return Object.keys(val).sort().reduce(function (a, key) { a[key] = val[key]; return a; }, {});
      }
      return val;
    });
  }
  function local(key, val) { // write to localStorage without triggering a cloud push, then refresh React state
    mute = true;
    try { rawSet.call(localStorage, key, JSON.stringify(val)); } finally { mute = false; }
    window.dispatchEvent(new CustomEvent('webtrex_storage_sync', { detail: { key: key } }));
  }
  function note(msg, force) {
    console.info('[webtrex-cloud]', msg);
    if (!force && !admin()) return; // visitors never see sync messages
    if (!pill) {
      pill = document.createElement('div');
      pill.style.cssText = 'position:fixed;left:16px;bottom:16px;z-index:200;max-width:340px;padding:10px 14px;border-radius:14px;background:#0A0A0A;color:#fff;font:600 12px/1.4 system-ui,sans-serif;box-shadow:0 10px 30px rgba(0,0,0,.25);transition:opacity .3s';
      (document.body || document.documentElement).appendChild(pill);
    }
    pill.textContent = msg;
    pill.style.opacity = 1;
    clearTimeout(pill._t);
    pill._t = setTimeout(function () { pill.style.opacity = 0; }, 6500);
  }
  function fail(e) {
    note('Cloud: ' + (e.code || e.message) + (e.code === 'permission-denied' ? ' (publish the Firestore rules)' : ''));
  }

  async function upload(dataUrl, name) {
    if (!storage) return dataUrl;
    try {
      var r = storage.ref('uploads/' + Date.now() + '-' + Math.random().toString(36).slice(2, 7) + '-' + String(name || 'file').replace(/[^\w.-]+/g, '_'));
      await r.putString(dataUrl, 'data_url');
      return await r.getDownloadURL();
    } catch (e) {
      note('Storage upload failed: ' + (e.code || e.message));
      return dataUrl;
    }
  }

  async function lift(p) { // move big inline images/files into Storage so Firestore docs stay under 1 MB
    var up = function (u, n) { return typeof u === 'string' && u.indexOf('data:') === 0 && u.length > 20000 ? upload(u, n) : u; };
    var o = Object.assign({}, p);
    o.thumbnail = await up(p.thumbnail, p.title);
    o.images = await Promise.all((p.images || []).map(function (u) { return up(u, p.title); }));
    o.files = await Promise.all((p.files || []).map(async function (f) {
      return Object.assign({}, f, { dataUrl: await up(f.dataUrl, f.name) });
    }));
    return o;
  }

  async function push(t, val) {
    try {
      var A = admin();
      if (t === 's') { if (A) await db.collection('settings').doc('site').set(clean(val)); return; }
      var col = db.collection(t === 'p' ? 'projects' : 'messages');
      if (!A) { // visitors can only create contact messages
        if (t !== 'm') return;
        for (var i = 0; i < val.length; i++) {
          var m = val[i];
          if (!sent[m.id] && !snap.m.some(function (x) { return x.id === m.id; })) { sent[m.id] = 1; await col.doc(m.id).set(clean(m)); }
        }
        return;
      }
      if (t === 'p') {
        var lifted = await Promise.all(val.map(lift));
        if (sj(lifted) !== sj(val)) local(K.p, lifted);
        val = lifted;
      }
      var prev = snap[t], keep = {}, b = db.batch();
      val.forEach(function (x) {
        keep[x.id] = 1;
        var o = prev.find(function (p) { return p.id === x.id; });
        if (!o || sj(o) !== sj(x)) b.set(col.doc(x.id), clean(x));
      });
      prev.forEach(function (p) { if (!keep[p.id]) b.delete(col.doc(p.id)); });
      await b.commit();
    } catch (e) { fail(e); }
  }

  // Mirror every app write (projects / messages / settings) to the cloud
  Storage.prototype.setItem = function (k, v) {
    rawSet.call(this, k, v);
    if (mute || this !== localStorage) return;
    var t = k === K.p ? 'p' : k === K.m ? 'm' : k === K.s ? 's' : null;
    if (t) setTimeout(function () { try { push(t, JSON.parse(v)); } catch (e) {} }, 0);
  };

  // Live data: visitors get published projects, the admin gets everything
  function listen() {
    subs.forEach(function (u) { u(); });
    subs = [];
    var isA = admin(), pc = db.collection('projects');
    subs.push((isA ? pc : pc.where('status', '==', 'published')).onSnapshot(function (s) {
      var list = s.docs.map(function (d) { return d.data(); });
      var loc = JSON.parse(localStorage.getItem(K.p) || '[]');
      if (isA && s.empty && loc.length) { push('p', loc); return; } // first run: migrate local projects up
      snap.p = list;
      local(K.p, list);
    }, fail));
    subs.push(db.collection('settings').doc('site').onSnapshot(function (d) { if (d.exists) local(K.s, d.data()); }, fail));
    if (isA) {
      subs.push(db.collection('messages').onSnapshot(function (s) {
        var list = s.docs.map(function (d) { return d.data(); }).sort(function (a, b) { return String(b.createdAt).localeCompare(String(a.createdAt)); });
        snap.m = list;
        local(K.m, list);
      }, fail));
    }
  }
  auth.onAuthStateChanged(listen);

  // Admin sign-in / password change / logout also drive the real Firebase account
  document.addEventListener('submit', function (e) {
    var f = e.target;
    if (!f || !f.querySelector) return;
    var em = f.querySelector('input[type=email]');
    var pw = f.querySelector('input[placeholder="\u2022\u2022\u2022\u2022\u2022\u2022\u2022\u2022\u2022\u2022\u2022\u2022"]');
    var pws = f.querySelectorAll('input[type=password]');
    if (em && pw && em.value.trim().toLowerCase() === ADMIN) {
      auth.signInWithEmailAndPassword(ADMIN, pw.value.trim()).then(
        function () { note('Cloud sync is live', true); },
        function (er) { note('Cloud login: ' + (er.code || er.message), true); }
      );
    } else if (pws.length === 3 && admin()) {
      var cur = pws[0].value, nw = pws[1].value;
      if (nw.length >= 8 && nw === pws[2].value) {
        auth.currentUser.reauthenticateWithCredential(firebase.auth.EmailAuthProvider.credential(ADMIN, cur))
          .then(function () { return auth.currentUser.updatePassword(nw); })
          .catch(function (er) { note('Cloud password: ' + (er.code || er.message), true); });
      }
    }
  }, true);
  document.addEventListener('click', function (e) {
    var b = e.target.closest && e.target.closest('button');
    if (b && /^\s*Logout\s*$/.test(b.textContent)) auth.signOut();
  }, true);

  // Real file storage: files over 650 KB used to be swapped for a text placeholder, upload the actual file instead
  window.addEventListener('load', function () {
    var orig = window.readFileAsDataUrl;
    if (typeof orig !== 'function') { console.warn('[webtrex-cloud] readFileAsDataUrl not found; large files keep the old placeholder behaviour'); return; }
    window.readFileAsDataUrl = async function (file) {
      if (storage && admin() && file.size > 650 * 1024) {
        try {
          var r = storage.ref('uploads/' + Date.now() + '-' + file.name.replace(/[^\w.-]+/g, '_'));
          await r.put(file);
          return await r.getDownloadURL();
        } catch (e) { note('Storage upload failed: ' + (e.code || e.message)); }
      }
      return orig(file);
    };
  });
})();
