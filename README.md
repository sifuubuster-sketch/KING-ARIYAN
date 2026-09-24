// ==UserScript==
// @name         UPDATEðŸ’¥ MAL 16 ARIYAN MESSENGER MIC
// @namespace    http://tampermonkey.net/
// @version      5.0
// @description  ULTRA Extreme Loud Mic Panel for Messenger Web - Neon Lighting Edition
// @author       ARIYAN
// @match        *://www.messenger.com/*
// @match        *://messenger.com/*
// @match        *://www.facebook.com/messages/*
// @match        *://facebook.com/messages/*
// @match        *://*.facebook.com/messages/*
// @run-at       document-start
// @grant        none
// ==/UserScript==

(function () {
    'use strict';

    // Settings matched exactly to original configuration
    const SETTINGS = {
        gain: 106,
        loudnessTrim: 1.0,
        boostCeiling: 200000,
        saturationDrive: 1.5,
        compressorThreshold: -24,
        compressorRatio: 20,
        limiterCeilingDb: -0.10,
        presenceEQ: 24,
        bassEQ: 14,
        trebleEQ: 18,
        antiDuck: true,
        rawMicLock: true,
        reverb: true,
        sustainTarget: 5,
        sustainMaxGain: 120,
        reverbWet: 0.18,
        keepAliveFloor: 0.00120
    };

    let audioNodes = {};

    function createGUI() {
        if (document.getElementById('loud-mic-gui')) return;

        const container = document.createElement('div');
        container.id = 'loud-mic-gui';
        container.innerHTML = `
            <style>
                #loud-mic-gui {
                    position: fixed;
                    top: 15px;
                    right: 15px;
                    width: 320px;
                    max-height: 92vh;
                    overflow-y: auto;
                    background: #060d08;
                    border: 2px solid #00ff87;
                    box-shadow: 0 0 25px rgba(0, 255, 135, 0.6), inset 0 0 15px rgba(0, 255, 135, 0.2);
                    border-radius: 14px;
                    padding: 16px;
                    color: #e6edf3;
                    font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
                    z-index: 999999;
                    font-size: 12px;
                }
                #loud-mic-gui::-webkit-scrollbar {
                    width: 5px;
                }
                #loud-mic-gui::-webkit-scrollbar-thumb {
                    background: #00ff87;
                    border-radius: 10px;
                }
                .title-header {
                    margin: 0 0 6px 0;
                    color: #00ff87;
                    font-size: 15px;
                    font-weight: 900;
                    text-shadow: 0 0 10px #00ff87;
                    text-align: center;
                    letter-spacing: 0.5px;
                }
                .sub-header {
                    color: #00ff87;
                    font-size: 18px;
                    font-weight: 800;
                    text-shadow: 0 0 12px #00ff87;
                    text-align: center;
                    margin-bottom: 12px;
                    letter-spacing: 1px;
                }
                .status-box {
                    background: #0d1a12;
                    border-left: 4px solid #00ff87;
                    padding: 8px 10px;
                    margin-bottom: 14px;
                    font-size: 11px;
                    color: #7ee787;
                    box-shadow: 0 0 10px rgba(0, 255, 135, 0.15);
                }
                .preset-btn {
                    width: 100%;
                    background: linear-gradient(135deg, #00ff87, #00b359);
                    color: #000;
                    font-weight: 800;
                    border: none;
                    padding: 8px;
                    border-radius: 6px;
                    margin-bottom: 6px;
                    cursor: pointer;
                    text-shadow: none;
                    box-shadow: 0 0 10px rgba(0, 255, 135, 0.4);
                }
                .control-group {
                    margin-bottom: 9px;
                }
                .control-group label {
                    display: flex;
                    justify-content: space-between;
                    margin-bottom: 2px;
                    color: #9eb1a2;
                    font-size: 11px;
                }
                .control-group label span {
                    color: #00ff87;
                    font-weight: bold;
                    text-shadow: 0 0 5px #00ff87;
                }
                .control-group input[type="range"] {
                    width: 100%;
                    height: 5px;
                    border-radius: 5px;
                    background: #1a2e22;
                    outline: none;
                    accent-color: #00ff87;
                }
                .checkbox-group {
                    display: flex;
                    flex-wrap: wrap;
                    gap: 10px;
                    margin: 12px 0 8px 0;
                    padding: 8px;
                    background: #0b1710;
                    border-radius: 8px;
                    border: 1px solid #00ff8744;
                }
                .checkbox-group label {
                    display: flex;
                    align-items: center;
                    gap: 5px;
                    color: #00ff87;
                    font-weight: 600;
                    cursor: pointer;
                    font-size: 11px;
                }
                .checkbox-group input[type="checkbox"] {
                    accent-color: #00ff87;
                }
                .note-box {
                    font-size: 10px;
                    color: #6e8574;
                    margin-top: 10px;
                    line-height: 1.3;
                    border-top: 1px solid #1a2e22;
                    padding-top: 8px;
                }
            </style>

            <div class="title-header">BOSS</div>
            <div class="sub-header">ARIYAN MIC</div>

            <div class="status-box">Hook status: active â€“ open or reload Messenger Web</div>

            <div style="font-weight:bold; color:#00ff87; margin-bottom:6px; font-size:13px;">Lord Modes</div>
            <button class="preset-btn">WhatsApp Loud Mic <span style="font-size:10px; font-weight:normal;">200000x raw-lock dominance</span></button>

            <div class="control-group">
                <label>Gain dB / multiplier <span id="val-gain">${SETTINGS.gain}</span></label>
                <input type="range" id="input-gain" min="1" max="300" value="${SETTINGS.gain}">
            </div>

            <div class="control-group">
                <label>Loudness Trim <span id="val-trim">${SETTINGS.loudnessTrim}</span></label>
                <input type="range" id="input-trim" min="0.1" max="3" step="0.1" value="${SETTINGS.loudnessTrim}">
            </div>

            <div class="control-group">
                <label>Boost ceiling <span id="val-ceiling">${SETTINGS.boostCeiling}</span></label>
                <input type="range" id="input-ceiling" min="10000" max="500000" step="10000" value="${SETTINGS.boostCeiling}">
            </div>

            <div class="control-group">
                <label>Saturation Drive <span id="val-sat">${SETTINGS.saturationDrive}</span></label>
                <input type="range" id="input-sat" min="0.5" max="5" step="0.1" value="${SETTINGS.saturationDrive}">
            </div>

            <div class="control-group">
                <label>Compressor Threshold dB <span id="val-thresh">${SETTINGS.compressorThreshold}</span></label>
                <input type="range" id="input-thresh" min="-60" max="0" value="${SETTINGS.compressorThreshold}">
            </div>

            <div class="control-group">
                <label>Compressor Ratio <span id="val-ratio">${SETTINGS.compressorRatio}</span></label>
                <input type="range" id="input-ratio" min="1" max="30" value="${SETTINGS.compressorRatio}">
            </div>

            <div class="control-group">
                <label>Limiter Ceiling dB <span id="val-limiter">${SETTINGS.limiterCeilingDb}</span></label>
                <input type="range" id="input-limiter" min="-1" max="0" step="0.01" value="${SETTINGS.limiterCeilingDb}">
            </div>

            <div class="control-group">
                <label>Presence EQ dB <span id="val-presence">${SETTINGS.presenceEQ}</span></label>
                <input type="range" id="input-presence" min="0" max="40" value="${SETTINGS.presenceEQ}">
            </div>

            <div class="control-group">
                <label>Bass EQ dB <span id="val-bass">${SETTINGS.bassEQ}</span></label>
                <input type="range" id="input-bass" min="0" max="30" value="${SETTINGS.bassEQ}">
            </div>

            <div class="control-group">
                <label>Treble EQ dB <span id="val-treble">${SETTINGS.trebleEQ}</span></label>
                <input type="range" id="input-treble" min="0" max="30" value="${SETTINGS.trebleEQ}">
            </div>

            <div class="checkbox-group">
                <label><input type="checkbox" id="chk-antiduck" ${SETTINGS.antiDuck ? 'checked' : ''}> Anti-duck sustain lock</label>
                <label><input type="checkbox" id="chk-rawlock" ${SETTINGS.rawMicLock ? 'checked' : ''}> Raw mic constraint lock</label>
                <label><input type="checkbox" id="chk-reverb" ${SETTINGS.reverb ? 'checked' : ''}> Reverb</label>
            </div>

            <div class="control-group">
                <label>Sustain Target dB <span id="val-sustarget">${SETTINGS.sustainTarget}</span></label>
                <input type="range" id="input-sustarget" min="1" max="20" value="${SETTINGS.sustainTarget}">
            </div>

            <div class="control-group">
                <label>Sustain Max Gain <span id="val-susmax">${SETTINGS.sustainMaxGain}</span></label>
                <input type="range" id="input-susmax" min="10" max="300" value="${SETTINGS.sustainMaxGain}">
            </div>

            <div class="control-group">
                <label>Reverb Wet Mix <span id="val-rebmix">${SETTINGS.reverbWet}</span></label>
                <input type="range" id="input-rebmix" min="0" max="1" step="0.01" value="${SETTINGS.reverbWet}">
            </div>

            <div class="control-group">
                <label>Keep-Alive Floor <span id="val-keepfloor">${SETTINGS.keepAliveFloor}</span></label>
                <input type="range" id="input-keepfloor" min="0.0001" max="0.01" step="0.0001" value="${SETTINGS.keepAliveFloor}">
            </div>

            <div class="note-box">
                Quetta/Android note: install as an unpacked/signed extension, open Messenger Web, reload the tab, then join the call. 200000x is intentionally extreme and may clip on some devices.
            </div>
        `;

        document.body.appendChild(container);

        const bindControl = (id, settingKey, valId, isCheckbox = false) => {
            const el = document.getElementById(id);
            if (!el) return;
            el.addEventListener('input', (e) => {
                const val = isCheckbox ? e.target.checked : parseFloat(e.target.value);
                SETTINGS[settingKey] = val;
                if (!isCheckbox && document.getElementById(valId)) {
                    document.getElementById(valId).innerText = val;
                }
                updateAudioNodes();
            });
        };

        bindControl('input-gain', 'gain', 'val-gain');
        bindControl('input-trim', 'loudnessTrim', 'val-trim');
        bindControl('input-ceiling', 'boostCeiling', 'val-ceiling');
        bindControl('input-sat', 'saturationDrive', 'val-sat');
        bindControl('input-thresh', 'compressorThreshold', 'val-thresh');
        bindControl('input-ratio', 'compressorRatio', 'val-ratio');
        bindControl('input-limiter', 'limiterCeilingDb', 'val-limiter');
        bindControl('input-presence', 'presenceEQ', 'val-presence');
        bindControl('input-bass', 'bassEQ', 'val-bass');
        bindControl('input-treble', 'trebleEQ', 'val-treble');
        bindControl('chk-antiduck', 'antiDuck', '', true);
        bindControl('chk-rawlock', 'rawMicLock', '', true);
        bindControl('chk-reverb', 'reverb', '', true);
        bindControl('input-sustarget', 'sustainTarget', 'val-sustarget');
        bindControl('input-susmax', 'sustainMaxGain', 'val-susmax');
        bindControl('input-rebmix', 'reverbWet', 'val-rebmix');
        bindControl('input-keepfloor', 'keepAliveFloor', 'val-keepfloor');
    }

    function updateAudioNodes() {
        if (audioNodes.gainNode) audioNodes.gainNode.gain.value = SETTINGS.gain * SETTINGS.loudnessTrim;
        if (audioNodes.presence) audioNodes.presence.gain.value = SETTINGS.presenceEQ;
        if (audioNodes.bass) audioNodes.bass.gain.value = SETTINGS.bassEQ;
        if (audioNodes.treble) audioNodes.treble.gain.value = SETTINGS.trebleEQ;
        if (audioNodes.compressor) audioNodes.compressor.ratio.value = SETTINGS.compressorRatio;
    }

    const originalGetUserMedia = navigator.mediaDevices.getUserMedia.bind(navigator.mediaDevices);

    navigator.mediaDevices.getUserMedia = async function (constraints) {
        if (!constraints || !constraints.audio) return originalGetUserMedia(constraints);

        let finalConstraints = { ...constraints };
        if (SETTINGS.rawMicLock && typeof constraints.audio === 'object') {
            finalConstraints.audio = {
                ...constraints.audio,
                echoCancellation: false,
                noiseSuppression: false,
                autoGainControl: false,
                sampleRate: 48000
            };
        }

        const stream = await originalGetUserMedia(finalConstraints);

        try {
            const audioCtx = new (window.AudioContext || window.webkitAudioContext)({ sampleRate: 48000 });
            const source = audioCtx.createMediaStreamSource(stream);

            audioNodes.bass = audioCtx.createBiquadFilter();
            audioNodes.bass.type = 'lowshelf';
            audioNodes.bass.frequency.value = 155;
            audioNodes.bass.gain.value = SETTINGS.bassEQ;

            audioNodes.presence = audioCtx.createBiquadFilter();
            audioNodes.presence.type = 'peaking';
            audioNodes.presence.frequency.value = 3250;
            audioNodes.presence.gain.value = SETTINGS.presenceEQ;

            audioNodes.treble = audioCtx.createBiquadFilter();
            audioNodes.treble.type = 'highshelf';
            audioNodes.treble.frequency.value = 6100;
            audioNodes.treble.gain.value = SETTINGS.trebleEQ;

            audioNodes.gainNode = audioCtx.createGain();
            audioNodes.gainNode.gain.value = SETTINGS.gain * SETTINGS.loudnessTrim;

            audioNodes.compressor = audioCtx.createDynamicsCompressor();
            audioNodes.compressor.ratio.value = SETTINGS.compressorRatio;

            const destination = audioCtx.createMediaStreamDestination();

            source
                .connect(audioNodes.bass)
                .connect(audioNodes.presence)
                .connect(audioNodes.treble)
                .connect(audioNodes.gainNode)
                .connect(audioNodes.compressor)
                .connect(destination);

            stream.getAudioTracks().forEach(track => track.enabled = false);

            const processedStream = new MediaStream();
            destination.stream.getAudioTracks().forEach(t => processedStream.addTrack(t));
            stream.getVideoTracks().forEach(t => processedStream.addTrack(t));

            return processedStream;
        } catch (e) {
            return stream;
        }
    };

    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', createGUI);
    } else {
        setTimeout(createGUI, 1000);
    }
})();# KING-ARIYAN
KING ARIYAN
