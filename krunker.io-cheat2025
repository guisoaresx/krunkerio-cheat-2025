// ==UserScript==
// @name         KRUNKER - GuiSoares_X - Finalizado
// @namespace    http://tampermonkey.net/
// @version      0.4.3
// @description  ESP + Aimbot + Chams + TriggerBot + NoRecoil + BunnyHop + HUD (Hotkeys: B/L/T/G/F/M/N/K/C/P/J/V/O/H)
// @author       IG: GuiSoares_x (zTheMonio/Deus²²)
// @match        *://krunker.io/*
// @match        *://browserfps.com/*
// @exclude      *://krunker.io/social*
// @exclude      *://krunker.io/editor*
// @icon         https://www.google.com/s2/favicons?domain=krunker.io
// @grant        none
// @run-at       document-start
// @require      https://unpkg.com/three@0.150.0/build/three.min.js
// ==/UserScript==

const THREE = window.THREE;
delete window.THREE;

const settings = {
    aimbotEnabled: true,
    aimbotTarget: 'body',
    aimbotSmoothness: 0.5,
    aimFov: 45,
    aimbotOnRightMouse: false,
    espEnabled: false,
    espLines: false,
    wireframe: false,
    chams: false,
    chamsEnemy: 0xff00cc,
    chamsAlly: 0x00ff88,
    triggerBot: false,
    triggerThreshold: 6,
    recoilComp: false,
    recoilCompFactor: 0.7,
    autoBhop: false
};

const aimbotTargets = ['body', 'nut', 'head'];

const keyToSetting = {
    KeyB: 'aimbotEnabled',
    KeyL: 'aimbotOnRightMouse',
    KeyM: 'espEnabled',
    KeyN: 'espLines',
    KeyK: 'wireframe',
    KeyC: 'chams',
    KeyP: 'recoilComp',
    KeyJ: 'autoBhop',
    KeyV: 'triggerBot'
};

let gui = createGUI();
let scene;

const x = {
    window: window,
    document: document,
    querySelector: document.querySelector,
    consoleLog: console.log,
    ReflectApply: Reflect.apply,
    ArrayPrototype: Array.prototype,
    ArrayPush: Array.prototype.push,
    ObjectPrototype: Object.prototype,
    clearInterval: window.clearInterval,
    setTimeout: window.setTimeout,
    reToString: RegExp.prototype.toString,
    indexOf: String.prototype.indexOf,
    requestAnimationFrame: window.requestAnimationFrame
};

x.consoleLog('Waiting to inject...');

const proxied = function(object) {
    try {
        if (typeof object === 'object' &&
            typeof object.parent === 'object' &&
            object.parent.type === 'Scene' &&
            object.parent.name === 'Main') {

            x.consoleLog('Found Scene!')
            scene = object.parent;
            x.ArrayPrototype.push = x.ArrayPush;
        }
    } catch (error) {}
    return x.ArrayPush.apply(this, arguments);
}

const tempVector = new THREE.Vector3();
const tempObject = new THREE.Object3D();
tempObject.rotation.order = 'YXZ';

const geometry = new THREE.EdgesGeometry(new THREE.BoxGeometry(5, 15, 5).translate(0, 7.5, 0));

const material = new THREE.RawShaderMaterial({
    vertexShader: `
    	attribute vec3 position;
    	uniform mat4 projectionMatrix;
    	uniform mat4 modelViewMatrix;
    	void main() {
    		gl_Position = projectionMatrix * modelViewMatrix * vec4( position, 1.0 );
    		gl_Position.z = 1.0;
    	}
    	`,
    fragmentShader: `
    	void main() {
    		gl_FragColor = vec4( 1.0, 0.0, 0.0, 1.0 );
    	}
    	`
});

const line = new THREE.LineSegments(new THREE.BufferGeometry(), material.clone());
line.frustumCulled = false;

const linePositions = new THREE.BufferAttribute(new Float32Array(100 * 2 * 3), 3);
line.geometry.setAttribute('position', linePositions);

let injectTimer = null;
let rightMouseDown = false;
let spaceHeld = false;
let bhopTimer = null;

function lerp(start, end, t) {
    return start * (1 - t) + end * t;
}

function getGameCanvas() {
    const cands = document.getElementsByTagName('canvas');
    for (let i = 0; i < cands.length; i++) {
        const c = cands[i];
        if (c.width > 50 && c.height > 50 && c.style.display !== 'none') return c;
    }
    return document.querySelector('canvas');
}

function triggerShot() {
    const cvs = getGameCanvas();
    const cx = Math.floor(window.innerWidth / 2);
    const cy = Math.floor(window.innerHeight / 2);
    if (cvs) {
        try {
            const down = new PointerEvent('pointerdown', { bubbles: true, cancelable: true, pointerType: 'mouse', isPrimary: true, buttons: 1, clientX: cx, clientY: cy });
            const up = new PointerEvent('pointerup', { bubbles: true, cancelable: true, pointerType: 'mouse', isPrimary: true, buttons: 0, clientX: cx, clientY: cy });
            cvs.dispatchEvent(down);
            setTimeout(() => cvs.dispatchEvent(up), 10);
        } catch (e) {
            try {
                document.dispatchEvent(new MouseEvent('mousedown', { bubbles: true, cancelable: true, button: 0 }));
                setTimeout(() => document.dispatchEvent(new MouseEvent('mouseup', { bubbles: true, cancelable: true, button: 0 })), 10);
            } catch (e2) {}
        }
    }
}

function applyChamsToPlayer(player) {
    try {
        player.traverse(node => {
            if (!node || !node.material) return;
            try { node.material.wireframe = settings.wireframe; } catch (e) {}
            try { node.material.depthTest = !settings.chams ? true : false; } catch (e) {}
            try { node.material.depthWrite = !settings.chams ? true : false; } catch (e) {}
            try { node.renderOrder = settings.chams ? 9999 : 0; } catch (e) {}
            if (settings.chams) {
                try {
                    if (myPlayer && myPlayer.team !== undefined && player.team !== undefined && myPlayer.team === player.team) {
                        node.material.color && node.material.color.setHex(settings.chamsAlly);
                    } else {
                        node.material.color && node.material.color.setHex(settings.chamsEnemy);
                    }
                    try { node.material.transparent = true; node.material.opacity = 0.95; } catch (e) {}
                } catch (e) {}
            } else {
                try { node.material.transparent = false; node.material.opacity = 1; } catch (e) {}
            }
            try { node.material.needsUpdate = true; } catch (e) {}
        });
    } catch (e) {}
}

function tryZeroWeaponSpread() {
    try {
        if (!myPlayer) return;
        try {
            if (myPlayer.weapon) {
                if (settings.recoilComp) {
                    if (typeof myPlayer.weapon.recoil !== 'undefined') myPlayer.weapon.recoil = 0;
                    if (typeof myPlayer.weapon.spread !== 'undefined') myPlayer.weapon.spread = 0;
                }
            }
        } catch (e) {}
        try {
            const camRoot = myPlayer.children && myPlayer.children[0];
            if (camRoot && camRoot.weapon) {
                if (settings.recoilComp) {
                    if (typeof camRoot.weapon.recoil !== 'undefined') camRoot.weapon.recoil = 0;
                    if (typeof camRoot.weapon.spread !== 'undefined') camRoot.weapon.spread = 0;
                }
            }
        } catch (e) {}
        try {
            if (settings.recoilComp && myPlayer && myPlayer.children && myPlayer.children[0]) {
                const cam = myPlayer.children[0];
                cam.rotation.x = lerp(cam.rotation.x, 0, settings.recoilCompFactor * 0.45);
            }
        } catch (e) {}
    } catch (e) {}
}

function handleBhopState() {
    if (!settings.autoBhop) {
        if (bhopTimer) { clearInterval(bhopTimer); bhopTimer = null; }
        return;
    }
    if (!bhopTimer) {
        bhopTimer = setInterval(() => {
            try {
                const cvs = getGameCanvas();
                try { cvs && cvs.focus && cvs.focus(); } catch (e) {}
                const kd = new KeyboardEvent('keydown', {key: ' ', code: 'Space', bubbles: true});
                const ku = new KeyboardEvent('keyup', {key: ' ', code: 'Space', bubbles: true});
                window.dispatchEvent(kd);
                setTimeout(() => window.dispatchEvent(ku), 30);
            } catch (e) {}
        }, 160);
    }
}

const el = document.createElement('div');

el.innerHTML = `<style>
.dialog { position: absolute; left: 50%; top: 50%; padding: 22px; background: rgba(18,18,20,0.96); backdrop-filter: blur(6px); border: 1px solid rgba(255,255,255,0.04); border-radius: 12px; color: #fff; transform: translate(-50%, -50%); text-align: left; z-index: 999999; box-shadow: 0 10px 30px rgba(0,0,0,0.35); font-family: 'Segoe UI','Roboto','Arial',sans-serif; width: 460px; }
.dialog * { color: #fff; line-height: 1.6; }
.dialog h3 { text-align: center; margin-top:0; margin-bottom: 10px; font-size: 20px; color:#fff; }
.dialog h4 { margin: 12px 0 6px 0; border-bottom: 1px solid rgba(255,255,255,0.06); padding-bottom: 6px; color: #70b6ff; }
.dialog .key { background: rgba(255,255,255,0.06); padding: 2px 6px; border-radius: 4px; font-weight: bold; margin-right: 8px; font-family: 'Consolas', monospace; color:#000; }
.dialog .warning { text-align: center; background: rgba(231,76,60,0.08); border: 1px solid rgba(231,76,60,0.12); padding: 10px; border-radius: 5px; margin-bottom: 12px; color: #ffdede; }
.close { position: absolute; right: 6px; top: 6px; width: 18px; height: 18px; cursor: pointer; opacity: 0.9; }
.close:before, .close:after { content: ' '; position: absolute; left: 50%; top: 50%; width: 100%; height: 18%; transform: translate(-50%, -50%) rotate(-45deg); background: #fff; }
.close:after { transform: translate(-50%, -50%) rotate(45deg); }
.btn { cursor: pointer; padding: 8px 12px; background: #c0392b; border-radius: 6px; border: none; color: #fff; text-transform: uppercase; font-weight: 700; }
.zui { position: fixed; right: 15px; top: 15px; z-index: 999999; display: flex; flex-direction: column; font-family: 'Segoe UI','Roboto','Arial'; font-size: 14px; color: #fff; width: 320px; user-select: none; background: rgba(20,20,24,0.96); backdrop-filter: blur(6px); border-radius: 8px; border:1px solid rgba(255,255,255,0.03); overflow: hidden; box-shadow: 0 8px 30px rgba(0,0,0,0.35); }
.zui-header { padding: 12px 14px; display: flex; justify-content: space-between; align-items: center; background: rgba(255,255,255,0.02); cursor: pointer; }
.zui-header span { font-size: 15px; font-weight: 700; color: #fff; }
.zui-toggle-icon { transition: transform 0.25s ease; color:#ddd; }
.zui.open .zui-toggle-icon { transform: rotate(180deg); }
.zui-content { max-height: 0; opacity: 0; overflow: hidden; transition: max-height 0.35s ease-out, opacity 0.25s ease-out; border-top: 1px solid rgba(255,255,255,0.02); }
.zui.open .zui-content { max-height: 520px; opacity: 1; }
.zui-item { padding: 10px 14px; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid rgba(255,255,255,0.02); cursor: pointer; }
.zui-item:hover { background: rgba(255,255,255,0.02); }
.zui-item span { color: #ddd; font-size: 13.5px; }
.zui-item .zui-item-value { font-weight: 700; font-size: 13px; padding: 4px 8px; border-radius: 6px; background: rgba(255,255,255,0.03); color:#fff; }
.zui-item.text { justify-content:center; color:#bbb; padding: 10px; }
.gsm7-msg { position: fixed; left: 10px; bottom: 10px; z-index: 999999; background: rgba(0,0,0,0.6); color: #fff; padding: 10px 12px; border-radius: 6px; font-weight: 700; }
</style>
<div class="dialog" style="display:none;">
  <div class="close" onclick="this.parentNode.style.display='none';"></div>
  <h3>Ajuda - Deus²²</h3>
  <div class="warning"><strong>AVISO:</strong> O uso de scripts pode resultar em banimento. Use por sua conta e risco.</div>
  <h4>Controles Gerais</h4>
  <div><span class="key">H</span> Abrir/Fechar este painel de Ajuda</div>
  <div><span class="key">O</span> Abrir/Fechar o Painel de Controle</div>
  <h4>Aimbot</h4>
  <div><span class="key">B</span> Ativar / Desativar</div>
  <div><span class="key">L</span> Mirar apenas com Botão Direito</div>
  <div><span class="key">T</span> Alternar Alvo (Head/Body/Nut)</div>
  <div><span class="key">G / Shift+G</span> Suavização +/-</div>
  <div><span class="key">F / Shift+F</span> FOV +/-</div>
  <h4>Visuais (ESP)</h4>
  <div><span class="key">M</span> Ativar / Desativar ESP (Caixas)</div>
  <div><span class="key">N</span> Ativar / Desativar ESP (Linhas)</div>
  <div><span class="key">K</span> Wireframe ON/OFF</div>
  <div><span class="key">C</span> Chams ON/OFF</div>
  <div><span class="key">P</span> No Recoil ON/OFF</div>
  <div><span class="key">V</span> TriggerBot ON/OFF</div>
  <div><span class="key">J</span> AutoBhop ON/OFF</div>
  <div style="text-align:center;color:#aaa;margin-top:10px;">Desenvolvido por zTheMonio</div>
</div>
<div class="zui" style="display:none;">
  <div class="zui-header"><span>Painel de Controle [O]</span><span class="zui-toggle-icon">▼</span></div>
  <div class="zui-content"></div>
</div>`;

const msgEl = el.querySelector('.gsm7-msg');
const dialogEl = el.querySelector('.dialog');

window.addEventListener('DOMContentLoaded', function() {
    while (el.children.length > 0) {
        document.body.appendChild(el.children[0]);
    }
    gui.style.display = 'none';
    document.body.appendChild(gui);
    createGuiContent();
});

window.addEventListener('pointerdown', function(e) { if (e.button === 2) rightMouseDown = true; });
window.addEventListener('pointerup', function(e) { if (e.button === 2) rightMouseDown = false; });

window.addEventListener('keydown', function(e) {
    if (e.code === 'Space') { spaceHeld = true; if (settings.autoBhop) handleBhopState(); }
});
window.addEventListener('keyup', function(e) {
    if (e.code === 'Space') { spaceHeld = false; if (bhopTimer) { clearInterval(bhopTimer); bhopTimer = null; } }
});

window.addEventListener('keyup', function(event) {
    if (x.document.activeElement && x.document.activeElement.value !== undefined) return;

    if (keyToSetting[event.code]) {
        toggleSetting(keyToSetting[event.code]);
        return;
    }

    switch (event.code) {
        case 'KeyO':
            toggleElementVisibility(gui);
            break;
        case 'KeyH':
            toggleElementVisibility(dialogEl);
            break;
        case 'KeyF':
            if (event.shiftKey) {
                settings.aimFov = Math.max(10, settings.aimFov - 25);
            } else {
                settings.aimFov = Math.min(1000, settings.aimFov + 25);
            }
            showMsg('Raio Aimbot', settings.aimFov);
            break;
        case 'KeyG':
            if (event.shiftKey) {
                settings.aimbotSmoothness = Math.max(0.05, settings.aimbotSmoothness - 0.05);
            } else {
                settings.aimbotSmoothness = Math.min(1, settings.aimbotSmoothness + 0.05);
            }
            showMsg('Força Aimbot', settings.aimbotSmoothness.toFixed(2));
            break;
        case 'KeyT': {
            const currentIndex = aimbotTargets.indexOf(settings.aimbotTarget);
            const nextIndex = (currentIndex + 1) % aimbotTargets.length;
            settings.aimbotTarget = aimbotTargets[nextIndex];
            showMsg('Alvo Aimbot', settings.aimbotTarget.toUpperCase());
            break;
        }
    }
});

function toggleElementVisibility(el) {
    const isHidden = el.style.display === 'none' || getComputedStyle(el).display === 'none';
    if (isHidden) {
        if (el.classList.contains('zui')) {
            el.style.display = 'flex';
            el.classList.add('open');
        } else {
            el.style.display = 'block';
        }
    } else {
        el.style.display = 'none';
        if (el.classList.contains('zui')) el.classList.remove('open');
    }
}

function showMsg(name, value) {
    const existing = document.querySelector('.gsm7-msg');
    if (existing) existing.remove();
    const msg = document.createElement('div');
    msg.className = 'gsm7-msg';
    msg.innerText = name + ': ' + (typeof value === 'boolean' ? (value ? 'ON' : 'OFF') : (typeof value === 'number' && !Number.isInteger(value) ? value.toFixed(2) : value));
    document.body.appendChild(msg);
    clearTimeout(msg._t);
    msg._t = setTimeout(() => { msg.remove(); }, 1600);
}

function createGuiContent() {
    const content = gui.querySelector('.zui-content');
    content.innerHTML = '';
    const settingToKey = {};
    for (const k in keyToSetting) settingToKey[keyToSetting[k]] = k;
    Object.keys(settings).forEach(prop => {
        let name = fromCamel(prop);
        let shortKey = settingToKey[prop] || (prop === 'aimbotTarget' ? 'T' : '');
        if (shortKey && shortKey.startsWith('Key')) shortKey = shortKey.slice(3);
        if (shortKey) name = `[${shortKey}] ${name}`;
        const item = fromHtml(`<div class="zui-item"><span>${name}</span><span class="zui-item-value" id="val_${prop}"></span></div>`);
        const valueEl = item.querySelector('.zui-item-value');
        const update = () => {
            const v = settings[prop];
            if (typeof v === 'boolean') { valueEl.innerText = v ? 'ON' : 'OFF'; valueEl.style.color = v ? '#7CFFB2' : '#FF8C8C'; }
            else if (typeof v === 'string') { valueEl.innerText = v.toUpperCase(); valueEl.style.color = '#CFEFFF'; }
            else { valueEl.innerText = Number.isInteger(v) ? v : v.toFixed(2); valueEl.style.color = '#fff'; }
        };
        if (typeof settings[prop] === 'boolean') {
            item.addEventListener('click', () => { settings[prop] = !settings[prop]; update(); showMsg(fromCamel(prop), settings[prop]); if (prop === 'autoBhop') handleBhopState(); });
        } else if (prop === 'aimbotTarget') {
            item.addEventListener('click', () => { const idx = (aimbotTargets.indexOf(settings.aimbotTarget) + 1) % aimbotTargets.length; settings.aimbotTarget = aimbotTargets[idx]; update(); showMsg('Alvo', settings.aimbotTarget.toUpperCase()); });
        } else item.classList.add('no-hover');
        update();
        content.appendChild(item);
        const p = `__${prop}`;
        settings[p] = settings[prop];
        Object.defineProperty(settings, prop, {
            get: () => settings[p],
            set(value) {
                if (settings[p] === value) return;
                settings[p] = value;
                update();
                if (typeof value === 'boolean' || typeof value === 'string') showMsg(fromCamel(prop), value);
                if (prop === 'autoBhop') handleBhopState();
            },
            enumerable: true,
            configurable: true
        });
    });
    content.appendChild(fromHtml(`<div class="zui-item text"><span>Desenvolvido por zTheMonio</span></div>`));
}

function createGUI() {
    const guiEl = fromHtml(`<div class="zui">
		<div class="zui-header">
			<span>Painel de Controle [O]</span>
			<span class="zui-toggle-icon">▼</span>
		</div>
		<div class="zui-content"></div>
	</div>`);
    const headerEl = guiEl.querySelector('.zui-header');
    headerEl.onclick = function() { guiEl.classList.toggle('open'); };
    return guiEl;
}

function fromCamel(text) {
    const result = text.replace(/([A-Z])/g, ' $1');
    return result.charAt(0).toUpperCase() + result.slice(1);
}

function fromHtml(html) {
    const div = document.createElement('div');
    div.innerHTML = html.trim();
    return div.firstChild;
}

function toggleSetting(key) {
    settings[key] = !settings[key];
    showMsg(fromCamel(key), settings[key]);
    const elv = document.getElementById('val_' + key);
    if (elv) elv.innerText = (typeof settings[key] === 'boolean') ? (settings[key] ? 'ON' : 'OFF') : settings[key];
    if (key === 'autoBhop') handleBhopState();
}

function animate() {
    x.requestAnimationFrame.call(x.window, animate);

    if (!scene && !injectTimer) {
        const el = x.querySelector.call(x.document, '#loadingBg');
        if (el && el.style.display === 'none') {
            x.consoleLog('Inject timer started!');
            injectTimer = x.setTimeout.call(x.window, () => {
                x.consoleLog('Injected!');
                x.ArrayPrototype.push = proxied;
            }, 2e3);
        }
    }

    if (scene === undefined || !scene.children) {
        return;
    }

    const players = [];
    let myPlayer;

    for (let i = 0; i < scene.children.length; i++) {
        const child = scene.children[i];
        if (child.type === 'Object3D') {
            try {
                if (child.children[0].children[0].type === 'PerspectiveCamera') {
                    myPlayer = child;
                } else {
                    players.push(child);
                }
            } catch (err) {}
        } else if (child.material) {
            child.material.wireframe = settings.wireframe;
        }
    }

    if (!myPlayer) {
        x.consoleLog('Player not found, finding new scene.');
        x.ArrayPrototype.push = proxied;
        return;
    }

    window.myPlayer = myPlayer;

    let counter = 0;
    let targetPlayer;
    let minDistance = Infinity;

    tempObject.matrix.copy(myPlayer.matrix).invert()

    for (let i = 0; i < scene.children.length; i++) {
        const child = scene.children[i];
        if (child.type === 'Object3D') {
            try {
                if (child.children[0].children[0].type === 'PerspectiveCamera') {} else {
                    if (child.team !== myPlayer.team || !child.ally) {
                        players.push(child);
                    }
                }
            } catch (err) {}
        }
    }

    for (let i = 0; i < players.length; i++) {
        const player = players[i];

        if (!player.lastPosition) {
            player.lastPosition = player.position.clone();
            player.velocity = new THREE.Vector3();
        }
        player.velocity.subVectors(player.position, player.lastPosition).multiplyScalar(0.9);
        player.lastPosition.copy(player.position);

        if (!player.box) {
            const box = new THREE.LineSegments(geometry, material.clone());
            box.frustumCulled = false;
            player.add(box);
            player.box = box;
        }

        if (player.position.x === myPlayer.position.x && player.position.z === myPlayer.position.z) {
            player.box.visible = false;
            if (line.parent !== player) {
                player.add(line);
            }
            continue;
        }

        linePositions.setXYZ(counter++, 0, 10, -5);
        tempVector.copy(player.position);
        tempVector.y += 9;
        tempVector.applyMatrix4(tempObject.matrix);
        linePositions.setXYZ(counter++, tempVector.x, tempVector.y, tempVector.z);

        player.visible = settings.espEnabled || player.visible;
        player.box.visible = settings.espEnabled;

        if (settings.chams) applyChamsToPlayer(player);
        else {
            try { player.traverse(n => { if (n && n.material) { n.material.depthTest = true; n.material.depthWrite = true; n.material.needsUpdate = true; } }); } catch (e) {}
        }

        const distance = player.position.distanceTo(myPlayer.position);
        if (distance < minDistance) {
            targetPlayer = player;
            minDistance = distance;
        }
    }

    linePositions.needsUpdate = true;
    line.geometry.setDrawRange(0, counter);
    line.visible = settings.espLines;

    tryZeroWeaponSpread();

    if (settings.aimbotEnabled === false || (settings.aimbotOnRightMouse && !rightMouseDown) || targetPlayer === undefined || !targetPlayer.visible) {
        return;
    }

    const targetPosition = new THREE.Vector3();
    switch (settings.aimbotTarget) {
        case 'head':
            try { targetPlayer.children[0].children[0].localToWorld(targetPosition); } catch (e) { targetPosition.setFromMatrixPosition(targetPlayer.matrixWorld); targetPosition.y += 10; }
            break;
        case 'nut':
            targetPosition.setFromMatrixPosition(targetPlayer.matrixWorld);
            targetPosition.y += 4;
            break;
        case 'body':
        default:
            targetPosition.setFromMatrixPosition(targetPlayer.matrixWorld);
            targetPosition.y += 7.5;
            break;
    }

    if (targetPlayer.velocity) {
        targetPosition.add(targetPlayer.velocity.clone().multiplyScalar(1.5));
    }

    const jitterAmount = 0.8;
    targetPosition.x += (Math.random() - 0.5) * jitterAmount;
    targetPosition.y += (Math.random() - 0.5) * jitterAmount * 0.7;
    targetPosition.z += (Math.random() - 0.5) * jitterAmount;

    const camera = myPlayer.children[0].children[0];
    const projectedPos = targetPosition.clone().project(camera);

    if (projectedPos.z > 1) {
        return;
    }

    const screenX = (projectedPos.x + 1) / 2 * window.innerWidth;
    const screenY = (1 - projectedPos.y) / 2 * window.innerHeight;

    const centerX = window.innerWidth / 2;
    const centerY = window.innerHeight / 2;
    const distance_from_center = Math.hypot(screenX - centerX, screenY - centerY);

    if (distance_from_center > settings.aimFov) {
        return;
    }

    tempObject.position.copy(myPlayer.position);
    tempObject.lookAt(targetPosition);

    const targetX = -tempObject.rotation.x;
    let targetY = tempObject.rotation.y + Math.PI;

    const currentY = myPlayer.rotation.y;
    let diff = targetY - currentY;

    if (diff > Math.PI) diff -= 2 * Math.PI;
    if (diff < -Math.PI) diff += 2 * Math.PI;

    const humanizedSmoothness = settings.aimbotSmoothness * (0.85 + Math.random() * 0.3);

    myPlayer.children[0].rotation.x = lerp(myPlayer.children[0].rotation.x, targetX, humanizedSmoothness);
    myPlayer.rotation.y += diff * humanizedSmoothness;

    if (settings.recoilComp && rightMouseDown) {
        try { myPlayer.children[0].rotation.x = myPlayer.children[0].rotation.x * (1 - settings.recoilCompFactor); } catch (e) {}
    }

    if (settings.triggerBot && distance_from_center < settings.triggerThreshold) {
        triggerShot();
    }
}

animate();
