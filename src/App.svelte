<script>
  import { onMount } from 'svelte';

  // --- LEVEL 0 & 1: CORE DEVICE STATE ---
  let isLocked = true;
  let isOpen = false;
  let isJammed = false;
  let pinInput = '';
  const masterPin = '1234';
  let pinFeedback = ''; // 'Access Granted', 'Incorrect PIN', etc.

  // --- SENSORS & INTERIOR DASHBOARD DATA ---
  let visitorPresent = false;
  let visitorName = 'Unknown Person';
  let packageCount = 3;
  let lastCourier = 'FedEx';
  let lastDeliveryTime = '2:15 PM';

  // --- OPTION 1: COMPLEX INPUTS (GUEST PASS MANAGER) ---
  let showGuestModal = false;
  let newGuestName = '';
  let newGuestPin = '';
  let newGuestWindow = '2:00 PM - 4:00 PM';
  let guestKeys = [
    { name: 'House Cleaner', pin: '8821', window: 'Fri 10:00 AM - 1:00 PM', active: true },
    { name: 'Dog Walker', pin: '5540', window: 'Mon-Fri 2:00 PM - 3:00 PM', active: true }
  ];

  // --- OPTION 2: SECONDARY DEVICE (MOCK SMARTPHONE) ---
  let phoneLocked = false;
  let phoneNotification = 'Front door is currently secure.';
  let phoneSliderProgress = 0;

  // --- OPTION 3: PROFILES & USAGE DATA ---
  let currentProfile = 'Working Professional';
  let accessHistory = [
    { time: '08:15 AM', type: 'Depart', user: 'Self' },
    { time: '11:42 AM', type: 'Delivery', user: 'FedEx' },
    { time: '02:15 PM', type: 'Delivery', user: 'UPS' },
    { time: '05:48 PM', type: 'Entry', user: 'Self' }
  ];
  let hourlyTraffic = [0, 0, 0, 0, 0, 0, 1, 3, 1, 0, 1, 2, 0, 0, 3, 2, 4, 2, 1, 1, 0, 0, 0, 0];

  const profiles = {
    'Working Professional': {
      traffic: [0, 0, 0, 0, 0, 0, 1, 4, 1, 0, 0, 1, 0, 0, 2, 1, 4, 3, 1, 0, 0, 0, 0, 0],
      packages: 2,
      courier: 'FedEx'
    },
    'Busy Family': {
      traffic: [0, 0, 0, 0, 0, 1, 2, 6, 2, 1, 3, 2, 2, 4, 5, 6, 7, 5, 3, 2, 1, 0, 0, 0],
      packages: 4,
      courier: 'Amazon Prime'
    },
    'Airbnb Host': {
      traffic: [0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 2, 3, 1, 2, 4, 2, 3, 2, 4, 2, 1, 0, 0, 0],
      packages: 0,
      courier: 'None'
    },
    'Vacation / Away': {
      traffic: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
      packages: 1,
      courier: 'USPS'
    }
  };

  // --- OPTION 4: TIME-LAPSE SIMULATION ---
  let isSimulating = false;
  let simHour = 8;
  let simTimer = null;

  // --- INFO MODAL (LEVEL 0 REQUIREMENT) ---
  let showInfoModal = false;

  // --- CONTROLS & LOGIC ---
  function enterDigit(d) {
    if (pinInput.length < 6) {
      pinInput += d;
    }
  }

  function clearPin() {
    pinInput = '';
    pinFeedback = '';
  }

  function submitPin() {
    if (isJammed) {
      pinFeedback = 'BOLT JAMMED';
      return;
    }
    const matchedGuest = guestKeys.find(g => g.pin === pinInput && g.active);
    if (pinInput === masterPin || matchedGuest) {
      const user = matchedGuest ? matchedGuest.name : 'Master User';
      unlockDoor(`PIN Authenticated (${user})`);
      pinFeedback = 'GRANTED';
    } else {
      pinFeedback = 'DENIED';
      triggerPhoneAlert('Failed PIN entry attempt detected at door.');
    }
    setTimeout(() => {
      pinInput = '';
      pinFeedback = '';
    }, 2000);
  }

  function scanFingerprint() {
    if (isJammed) {
      pinFeedback = 'BOLT JAMMED';
      return;
    }
    unlockDoor('Biometric Touch ID Verified');
    pinFeedback = 'TOUCH ID OK';
    setTimeout(() => (pinFeedback = ''), 2000);
  }

  function tapNFC() {
    if (isJammed) {
      pinFeedback = 'BOLT JAMMED';
      return;
    }
    unlockDoor('NFC Keycard / Phone BLE');
    pinFeedback = 'NFC OK';
    setTimeout(() => (pinFeedback = ''), 2000);
  }

  function unlockDoor(reason) {
    isLocked = false;
    logEvent(reason);
    triggerPhoneAlert(`Door unlocked via ${reason}`);
  }

  function lockDoor() {
    if (isOpen) {
      triggerPhoneAlert('Cannot lock: Door is physically open.');
      return;
    }
    isLocked = true;
    logEvent('Deadbolt engaged');
    triggerPhoneAlert('Door has been securely locked.');
  }

  function toggleOpen() {
    if (isLocked && !isOpen) {
      triggerPhoneAlert('Cannot open: Deadbolt is engaged.');
      return;
    }
    isOpen = !isOpen;
    logEvent(isOpen ? 'Door swung open' : 'Door closed into frame');
  }

  function toggleJam() {
    isJammed = !isJammed;
    if (isJammed) {
      isLocked = false;
      triggerPhoneAlert('ALERT: Deadbolt motor jammed! Latch misaligned.');
    } else {
      triggerPhoneAlert('Jam resolved: Bolt mechanism cleared.');
    }
  }

  function logEvent(type) {
    const now = new Date();
    const timeStr = `${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}`;
    accessHistory = [{ time: timeStr, type, user: 'Aayush' }, ...accessHistory.slice(0, 7)];
  }

  function triggerPhoneAlert(msg) {
    phoneNotification = msg;
  }

  function addGuestKey() {
    if (!newGuestName || !newGuestPin) return;
    guestKeys = [...guestKeys, { name: newGuestName, pin: newGuestPin, window: newGuestWindow, active: true }];
    newGuestName = '';
    newGuestPin = '';
    showGuestModal = false;
    logEvent('Created temporary guest PIN');
  }

  function setProfile(name) {
    currentProfile = name;
    hourlyTraffic = [...profiles[name].traffic];
    packageCount = profiles[name].packages;
    lastCourier = profiles[name].courier;
    logEvent(`Loaded scenario profile: ${name}`);
  }

  function toggleSimulation() {
    isSimulating = !isSimulating;
    if (isSimulating) {
      simTimer = setInterval(() => {
        simHour = (simHour + 1) % 24;
        if (simHour === 8) {
          isLocked = false;
          isOpen = true;
          logEvent('Sim: Resident leaving for work');
        } else if (simHour === 9) {
          isOpen = false;
          isLocked = true;
        } else if (simHour === 14) {
          packageCount += 1;
          lastCourier = 'Amazon';
          lastDeliveryTime = '2:00 PM';
          triggerPhoneAlert('Sim: Package detected on porch');
        } else if (simHour === 18) {
          isLocked = false;
          logEvent('Sim: Evening return');
        } else if (simHour === 22) {
          isLocked = true;
          logEvent('Sim: Night perimeter lockdown');
        }
      }, 900);
    } else {
      clearInterval(simTimer);
    }
  }
</script>

<main class="app-layout">
  <!-- ======================================================== -->
  <!-- LEFT REGION: DEVICE UI (MOCK SMART OBJECT - MULTI-SURFACE) -->
  <!-- ======================================================== -->
  <section class="device-viewport">
    <header class="section-badge">
      <span>DIGITAL SMART OBJECT: RESIDENTIAL ENTRYWAY</span>
      <span class="planes-tag">3 Interacting Physical Planes</span>
    </header>

    <div class="physical-door-assembly">
      <!-- 1. EXTERIOR SURFACE -->
      <div class="door-plane exterior-plane">
        <div class="plane-title">EXTERIOR FACE</div>

        <!-- Optical Camera & Peephole -->
        <div class="camera-hud">
          <div class="lens-dot {visitorPresent ? 'pulsing-lens' : ''}"></div>
          <div class="camera-feed-label">
            <small>[Live Footage]</small>
            <span>Camera Lens + Peephole Sensor</span>
          </div>
          {#if visitorPresent}
            <div class="visitor-tag">HUMAN DETECTED: {visitorName}</div>
          {/if}
        </div>

        <!-- Backlit Numeric Keypad -->
        <div class="keypad-chassis">
          <div class="keypad-header">
            <span class="nfc-badge">(( NFC Enabled ))</span>
            <div class="pin-display">
              {pinInput ? '•'.repeat(pinInput.length) : (pinFeedback || 'ENTER PIN')}
            </div>
          </div>

          <div class="keypad-matrix">
            {#each ['1','2','3','4','5','6','7','8','9'] as key}
              <button class="key-btn" on:click={() => enterDigit(key)}>{key}</button>
            {/each}
            <button class="key-btn clear" on:click={clearPin}>C</button>
            <button class="key-btn" on:click={() => enterDigit('0')}>0</button>
            <button class="key-btn enter" on:click={submitPin}>↵</button>
          </div>

          <!-- Integrated Fingerprint Sensor -->
          <div class="biometric-zone" on:click={scanFingerprint} title="Touch to authenticate">
            <div class="touch-icon">◉</div>
            <span>Fingerprint Sensor</span>
          </div>
        </div>

        <!-- Exterior Doorbell Chime -->
        <button
                class="doorbell-chime-btn"
                on:click={() => {
            visitorPresent = true;
            triggerPhoneAlert('Doorbell rang: Visitor at front porch.');
            logEvent('Doorbell Chime pressed');
          }}
        >
          <div class="bell-glow"></div>
          <span>🔔 RING DOORBELL</span>
        </button>
      </div>

      <!-- 2. NARROW DOOR EDGE (PERPENDICULAR SURFACE) -->
      <div class="door-plane edge-plane">
        <div class="plane-title">EDGE (1.75")</div>

        <div class="edge-hardware">
          <div class="alignment-sensor {isOpen ? 'unaligned' : 'aligned'}">
            <small>REED</small>
            <span>{isOpen ? 'AJAR' : 'FLUSH'}</span>
          </div>

          <!-- Physical Deadbolt Latch -->
          <div class="deadbolt-housing">
            <div
                    class="deadbolt-tongue
                {isLocked ? 'extended' : 'retracted'}
                {isJammed ? 'jammed-bolt' : ''}"
            ></div>
            <div class="bolt-label">
              {#if isJammed}
                <span class="bolt-status red">JAMMED</span>
              {:else if isLocked}
                <span class="bolt-status green">THROWN</span>
              {:else}
                <span class="bolt-status blue">RETRACTED</span>
              {/if}
            </div>
          </div>

          <!-- Edge Jam & Alignment LED Strip -->
          <div class="led-strip {isJammed ? 'led-jammed' : (isLocked ? 'led-locked' : 'led-open')}">
            <div class="led-segment"></div>
            <div class="led-segment"></div>
            <div class="led-segment"></div>
          </div>
        </div>
      </div>

      <!-- 3. INTERIOR SURFACE -->
      <div class="door-plane interior-plane">
        <div class="plane-title">INTERIOR DASHBOARD</div>

        <!-- Screen HUD -->
        <div class="interior-display">
          <!-- Glanceable Security Badge -->
          <div class="security-card {isJammed ? 'danger-card' : (isLocked ? 'secure-card' : 'warning-card')}">
            <div class="status-ring">
              <span class="status-symbol">{isJammed ? '⚠️' : (isLocked ? '🛡️' : '🔓')}</span>
            </div>
            <div class="status-text">
              <h3>{isJammed ? 'DEADBOLT JAMMED' : (isLocked ? '(SECURE)' : 'DOOR UNLOCKED')}</h3>
              <p>{isJammed ? 'Clear frame obstruction' : (isLocked ? 'All perimeters locked' : 'Immediate action recommended')}</p>
            </div>
          </div>

          <!-- Parcel Delivery Notification -->
          <div class="package-card">
            <div class="package-icon">📦</div>
            <div class="package-details">
              <strong>{packageCount} Packages left delivered</strong>
              <span>on porch at {lastDeliveryTime} [{lastCourier}]</span>
            </div>
          </div>

          <!-- Quick Navigation Actions -->
          <div class="interior-actions">
            <button class="nav-chip" on:click={() => showGuestModal = true}>
              🔑 Guest PINs ({guestKeys.length})
            </button>
            <button class="nav-chip" on:click={tapNFC}>
              📲 Mock NFC Tap
            </button>
          </div>
        </div>

        <!-- Physical Manual Thumbturn Slider -->
        <div class="physical-manual-latch">
          <div class="slider-title">PHYSICAL MANUAL LATCH</div>
          <div class="slider-track">
            <button
                    class="thumb-knob {isLocked ? 'knob-locked' : 'knob-unlocked'}"
                    on:click={() => isLocked ? unlockDoor('Manual Inside Slider') : lockDoor()}
            >
              {isLocked ? 'SLID: LOCKED' : 'SLID: UNLOCKED'}
            </button>
          </div>
          <div class="slider-mapping-label">
            ↑ Slide Up: Lock | ↓ Slide Down: Retract
          </div>
        </div>
      </div>
    </div>

    <!-- OPTION 1: MODAL DIALOG (GUEST PIN MANAGEMENT) -->
    {#if showGuestModal}
      <div class="modal-backdrop">
        <div class="modal-window">
          <h3>Guest Pass PIN Configuration</h3>
          <p>Create temporary, schedule-restricted credentials for service workers or guests.</p>
          <div class="form-group">
            <label>Guest Identifier</label>
            <input type="text" bind:value={newGuestName} placeholder="e.g., Courier / Sitter" />
          </div>
          <div class="form-group">
            <label>4-to-6 Digit Access PIN</label>
            <input type="password" maxlength="6" bind:value={newGuestPin} placeholder="4-6 Digits" />
          </div>
          <div class="form-group">
            <label>Active Recurring Window</label>
            <select bind:value={newGuestWindow}>
              <option value="One-Time Pass (Next 1 Hour)">One-Time Pass (Next 1 Hour)</option>
              <option value="Daily 8:00 AM - 12:00 PM">Daily 8:00 AM - 12:00 PM</option>
              <option value="Fri 10:00 AM - 1:00 PM">Fri 10:00 AM - 1:00 PM</option>
            </select>
          </div>
          <div class="modal-buttons">
            <button class="save-btn" on:click={addGuestKey}>Save & Deploy PIN</button>
            <button class="cancel-btn" on:click={() => showGuestModal = false}>Cancel</button>
          </div>
        </div>
      </div>
    {/if}
  </section>

  <!-- ======================================================== -->
  <!-- RIGHT REGION: TESTING HARNESS, CONTROLS & PROJECT INFO   -->
  <!-- ======================================================== -->
  <aside class="testing-viewport">
    <header class="project-header">
      <h2>AegisDoor Simulation Harness</h2>
      <p class="author-line">Solo Designer: <strong>Aayush Keshari</strong> | Svelte Mockup</p>
      <div class="header-links">
        <button class="info-pill-btn" on:click={() => showInfoModal = true}>ℹ️ Project & Test Guide</button>
      </div>
    </header>

    <!-- Graphic Placement Indicator -->
    <div class="diagram-card">
      <div class="diagram-tag">PHYSICAL SURFACE MAPPING</div>
      <div class="wireframe-graphic">
        <div class="wf-door">
          <div class="wf-ext">[Exterior Face]</div>
          <div class="wf-edge">[Edge: 1.75"]</div>
          <div class="wf-int">[Interior Dash]</div>
        </div>
      </div>
      <p class="diagram-desc">Controls reside across outside, inside, and perpendicular edge jambs.</p>
    </div>

    <!-- SENSOR TEST CONTROLS (LEVEL 0 REQUIREMENT) -->
    <section class="test-controls-group">
      <h3>Environment & Hardware Simulators</h3>
      <div class="btn-grid">
        <button class="sim-btn" on:click={() => { visitorPresent = !visitorPresent; visitorName = visitorPresent ? 'Delivery Agent' : 'None'; }}>
          {visitorPresent ? 'Clear Visitor Radar' : 'Trigger Approaching Visitor'}
        </button>

        <button class="sim-btn" on:click={toggleOpen}>
          {isOpen ? 'Slam Door Shut' : 'Physically Swing Door Open'}
        </button>

        <button class="sim-btn danger-sim" on:click={toggleJam}>
          {isJammed ? 'Clear Mechanical Jam' : 'Force Deadbolt Jam Obstruction'}
        </button>

        <button class="sim-btn" on:click={() => { packageCount += 1; lastCourier = 'UPS'; lastDeliveryTime = 'Just Now'; triggerPhoneAlert('New package dropped at doorstep.'); }}>
          Simulate Package Drop-off
        </button>
      </div>
    </section>

    <!-- OPTION 2: SECONDARY DEVICE (MOCK SMARTPHONE COMPANION) -->
    <section class="secondary-device-card">
      <div class="phone-frame">
        <div class="phone-speaker"></div>
        <div class="phone-screen">
          <div class="phone-time">14:22 · Companion iOS App</div>
          <div class="phone-push-card">
            <span class="push-app">AEGIS SHIELD NOTIFICATION</span>
            <p>{phoneNotification}</p>
          </div>
          <div class="phone-action">
            <button
                    class="phone-lock-toggle {isLocked ? 'btn-danger' : 'btn-primary'}"
                    on:click={() => isLocked ? unlockDoor('Mobile Phone App') : lockDoor()}
            >
              {isLocked ? 'Tap to Remote Unlock' : 'Tap to Remote Lock'}
            </button>
          </div>
        </div>
      </div>
    </section>

    <!-- OPTION 3: USER PROFILES & HISTORICAL DATA DISPLAY -->
    <section class="profile-data-card">
      <h3>Option 3: Sensor Data & User Scenarios</h3>
      <div class="profile-selector">
        {#each Object.keys(profiles) as name}
          <button
                  class="profile-chip {currentProfile === name ? 'active-profile' : ''}"
                  on:click={() => setProfile(name)}
          >
            {name}
          </button>
        {/each}
      </div>

      <!-- SVG Usage Histogram -->
      <div class="histogram-box">
        <div class="chart-label">Hourly Access Frequency (24h Activity)</div>
        <svg viewBox="0 0 240 60" class="bar-chart">
          {#each hourlyTraffic as val, i}
            <rect
                    x={i * 10 + 1}
                    y={60 - val * 8}
                    width="8"
                    height={val * 8}
                    fill={i === simHour && isSimulating ? '#38BDF8' : '#4F46E5'}
                    rx="1"
            />
          {/each}
        </svg>
        <div class="chart-axis">
          <span>12 AM</span>
          <span>6 AM</span>
          <span>12 PM</span>
          <span>6 PM</span>
          <span>11 PM</span>
        </div>
      </div>
    </section>

    <!-- OPTION 4: TIME-LAPSE ACCELERATED DAY SIMULATION -->
    <section class="simulation-group">
      <div class="sim-header">
        <h3>Option 4: 24h Accelerated Day Simulation</h3>
        <span class="sim-clock">Sim Time: {simHour.toString().padStart(2, '0')}:00</span>
      </div>
      <button class="sim-run-btn {isSimulating ? 'sim-active' : ''}" on:click={toggleSimulation}>
        {isSimulating ? '⏹ Pause Simulation' : '▶ Run 24h Day Cycle (1s = 1hr)'}
      </button>
    </section>
  </aside>

  <!-- INFO & TEST EXPLANATION MODAL (LEVEL 0) -->
  {#if showInfoModal}
    <div class="modal-backdrop">
      <div class="modal-window info-window">
        <h3>AegisDoor Simulation & Architecture Guide</h3>
        <p><strong>Non-Flat Distribution:</strong> The UI spans the <em>Exterior Face</em> (visitor chime, backlit keypad, camera lens, biometric sensor), the <em>Edge</em> (reed contact sensor, mechanical deadbolt throw, and jam LED), and the <em>Interior Face</em> (at-a-glance security ring, package alert card, and manual latch thumbturn).</p>
        <hr />
        <h4>Simulation Controls Guide:</h4>
        <ul>
          <li><strong>Numeric Keypad:</strong> Master PIN is <code>1234</code>. Type digits and press <code>↵</code>.</li>
          <li><strong>Force Deadbolt Jam:</strong> Simulates debris blocking the strike plate. The edge LED turns red and the interior ring displays a warning.</li>
          <li><strong>Approaching Visitor:</strong> Triggers the mmWave camera radar on the exterior unit.</li>
          <li><strong>Phone Companion:</strong> Demonstrates real-time state synchronization via a secondary mobile device.</li>
          <li><strong>24h Accelerated Time:</strong> Automatically advances the clock through morning commutes, midday couriers, and bedtime security lockouts.</li>
        </ul>
        <button class="save-btn" on:click={() => showInfoModal = false}>Close Guide</button>
      </div>
    </div>
  {/if}
</main>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
    background-color: #0F172A;
    color: #F8FAFC;
    box-sizing: border-box;
  }

  .app-layout {
    display: grid;
    grid-template-columns: 65% 35%;
    width: 100vw;
    height: 100vh;
    overflow: hidden;
  }

  /* LEFT PANEL: DEVICE VIEWPORT */
  .device-viewport {
    background: radial-gradient(circle at 50% 20%, #1E293B, #0F172A);
    border-right: 2px solid #334155;
    padding: 24px;
    display: flex;
    flex-direction: column;
    overflow-y: auto;
  }

  .section-badge {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: #1E293B;
    padding: 10px 18px;
    border-radius: 8px;
    border: 1px solid #475569;
    font-size: 0.82rem;
    font-weight: 700;
    letter-spacing: 0.05em;
    color: #94A3B8;
  }

  .planes-tag {
    background: #4F46E5;
    color: #FFFFFF;
    padding: 4px 10px;
    border-radius: 4px;
  }

  /* 3D DOOR SURFACE CONFIGURATION */
  .physical-door-assembly {
    display: grid;
    grid-template-columns: 1fr 100px 1fr;
    gap: 16px;
    margin-top: 20px;
    flex: 1;
    min-height: 600px;
  }

  .door-plane {
    background: #182234;
    border: 2px solid #334155;
    border-radius: 12px;
    padding: 16px;
    display: flex;
    flex-direction: column;
    position: relative;
    box-shadow: inset 0 2px 6px rgba(255,255,255,0.05), 0 8px 24px rgba(0,0,0,0.5);
  }

  .plane-title {
    font-size: 0.72rem;
    font-weight: 800;
    color: #64748B;
    letter-spacing: 0.1em;
    margin-bottom: 12px;
    text-align: center;
    border-bottom: 1px solid #334155;
    padding-bottom: 6px;
  }

  /* EXTERIOR PLANE */
  .camera-hud {
    background: #020617;
    border: 1px solid #1E293B;
    border-radius: 8px;
    padding: 12px;
    display: flex;
    align-items: center;
    gap: 12px;
    margin-bottom: 16px;
  }

  .lens-dot {
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background: #3B82F6;
    border: 3px solid #1D4ED8;
  }

  .pulsing-lens {
    background: #EF4444 !important;
    border-color: #991B1B !important;
    animation: pulse 1s infinite;
  }

  @keyframes pulse {
    0% { transform: scale(0.9); opacity: 0.8; }
    50% { transform: scale(1.15); opacity: 1; }
    100% { transform: scale(0.9); opacity: 0.8; }
  }

  .camera-feed-label {
    display: flex;
    flex-direction: column;
    font-size: 0.75rem;
  }

  .camera-feed-label small {
    color: #64748B;
  }

  .visitor-tag {
    margin-left: auto;
    background: #EF4444;
    color: white;
    font-size: 0.65rem;
    padding: 2px 6px;
    border-radius: 4px;
    font-weight: bold;
  }

  /* KEYPAD */
  .keypad-chassis {
    background: #0B1120;
    border: 1px solid #1E293B;
    border-radius: 10px;
    padding: 16px;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .keypad-header {
    width: 100%;
    text-align: center;
    margin-bottom: 12px;
  }

  .nfc-badge {
    font-size: 0.65rem;
    color: #38BDF8;
    letter-spacing: 0.05em;
  }

  .pin-display {
    background: #020617;
    border: 1px solid #334155;
    color: #38BDF8;
    height: 36px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.1rem;
    letter-spacing: 0.2em;
    font-family: monospace;
    border-radius: 6px;
    margin-top: 6px;
  }

  .keypad-matrix {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 8px;
    width: 100%;
    max-width: 180px;
  }

  .key-btn {
    background: #1E293B;
    border: 1px solid #334155;
    color: white;
    font-size: 1.1rem;
    font-weight: bold;
    height: 44px;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.1s;
  }

  .key-btn:hover {
    background: #334155;
  }

  .key-btn:active {
    background: #4F46E5;
    transform: scale(0.95);
  }

  .clear { color: #F87171; }
  .enter { color: #34D399; }

  .biometric-zone {
    margin-top: 14px;
    width: 100%;
    padding: 10px 0;
    border: 1px dashed #475569;
    border-radius: 8px;
    display: flex;
    flex-direction: column;
    align-items: center;
    cursor: pointer;
    background: #0F172A;
    transition: background 0.2s;
  }

  .biometric-zone:hover {
    background: #1E293B;
    border-color: #38BDF8;
  }

  .touch-icon {
    font-size: 1.4rem;
    color: #38BDF8;
  }

  .biometric-zone span {
    font-size: 0.68rem;
    color: #94A3B8;
    margin-top: 4px;
  }

  .doorbell-chime-btn {
    margin-top: auto;
    background: #1E293B;
    border: 2px solid #4F46E5;
    color: white;
    padding: 14px;
    border-radius: 10px;
    font-weight: 700;
    font-size: 0.85rem;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
  }

  .doorbell-chime-btn:hover {
    background: #4F46E5;
  }

  /* EDGE PLANE */
  .edge-plane {
    background: #0D1526;
    align-items: center;
  }

  .edge-hardware {
    display: flex;
    flex-direction: column;
    align-items: center;
    height: 100%;
    width: 100%;
    justify-content: space-around;
  }

  .alignment-sensor {
    background: #020617;
    border: 1px solid #334155;
    padding: 8px;
    border-radius: 6px;
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 80%;
  }

  .alignment-sensor small { font-size: 0.6rem; color: #64748B; }
  .alignment-sensor span { font-size: 0.75rem; font-weight: bold; }
  .aligned span { color: #10B981; }
  .unaligned span { color: #F59E0B; }

  .deadbolt-housing {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
  }

  .deadbolt-tongue {
    width: 48px;
    height: 36px;
    background: #94A3B8;
    border: 2px solid #CBD5E1;
    border-radius: 4px;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  }

  .retracted {
    transform: translateX(-20px);
    opacity: 0.3;
  }

  .extended {
    transform: translateX(0);
    opacity: 1;
  }

  .jammed-bolt {
    background: #EF4444 !important;
    border-color: #B91C1C !important;
    transform: translateX(-8px) rotate(4deg);
  }

  .bolt-label {
    margin-top: 8px;
    font-size: 0.68rem;
    font-weight: 800;
  }

  .green { color: #10B981; }
  .blue { color: #38BDF8; }
  .red { color: #EF4444; }

  .led-strip {
    width: 8px;
    height: 120px;
    border-radius: 4px;
    display: flex;
    flex-direction: column;
    gap: 4px;
  }

  .led-segment {
    flex: 1;
    border-radius: 2px;
  }

  .led-locked .led-segment { background: #10B981; box-shadow: 0 0 6px #10B981; }
  .led-open .led-segment { background: #38BDF8; box-shadow: 0 0 6px #38BDF8; }
  .led-jammed .led-segment { background: #EF4444; box-shadow: 0 0 8px #EF4444; animation: pulse 0.5s infinite; }

  /* INTERIOR PLANE */
  .interior-display {
    background: #020617;
    border: 1px solid #1E293B;
    border-radius: 10px;
    padding: 14px;
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  .security-card {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 12px;
    border-radius: 8px;
    border: 1px solid transparent;
  }

  .secure-card {
    background: rgba(16, 185, 129, 0.1);
    border-color: #10B981;
  }

  .warning-card {
    background: rgba(245, 158, 11, 0.1);
    border-color: #F59E0B;
  }

  .danger-card {
    background: rgba(239, 68, 68, 0.15);
    border-color: #EF4444;
  }

  .status-ring {
    font-size: 1.6rem;
  }

  .status-text h3 {
    margin: 0;
    font-size: 0.95rem;
    font-weight: 800;
  }

  .status-text p {
    margin: 2px 0 0 0;
    font-size: 0.72rem;
    color: #94A3B8;
  }

  .package-card {
    background: #0F172A;
    border: 1px solid #334155;
    border-radius: 8px;
    padding: 10px;
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .package-icon {
    font-size: 1.4rem;
  }

  .package-details {
    display: flex;
    flex-direction: column;
    font-size: 0.75rem;
  }

  .package-details span {
    color: #94A3B8;
  }

  .interior-actions {
    display: flex;
    gap: 8px;
  }

  .nav-chip {
    flex: 1;
    background: #1E293B;
    border: 1px solid #334155;
    color: #F8FAFC;
    padding: 8px;
    border-radius: 6px;
    font-size: 0.75rem;
    cursor: pointer;
  }

  .nav-chip:hover {
    background: #334155;
  }

  .physical-manual-latch {
    margin-top: auto;
    background: #0B1120;
    border: 1px solid #1E293B;
    border-radius: 10px;
    padding: 14px;
    text-align: center;
  }

  .slider-title {
    font-size: 0.65rem;
    font-weight: 700;
    color: #64748B;
    margin-bottom: 8px;
  }

  .slider-track {
    background: #020617;
    border: 1px solid #334155;
    border-radius: 20px;
    padding: 4px;
  }

  .thumb-knob {
    width: 100%;
    padding: 10px;
    border-radius: 16px;
    border: none;
    font-weight: 700;
    font-size: 0.8rem;
    cursor: pointer;
    transition: all 0.2s;
  }

  .knob-locked {
    background: #10B981;
    color: white;
  }

  .knob-unlocked {
    background: #3B82F6;
    color: white;
  }

  .slider-mapping-label {
    font-size: 0.65rem;
    color: #64748B;
    margin-top: 6px;
  }

  /* RIGHT PANEL: TESTING HARNESS */
  .testing-viewport {
    background: #0F172A;
    padding: 24px;
    display: flex;
    flex-direction: column;
    gap: 16px;
    overflow-y: auto;
  }

  .project-header h2 {
    margin: 0;
    font-size: 1.25rem;
    color: #F8FAFC;
  }

  .author-line {
    margin: 4px 0 8px 0;
    font-size: 0.8rem;
    color: #94A3B8;
  }

  .info-pill-btn {
    background: #1E293B;
    color: #38BDF8;
    border: 1px solid #38BDF8;
    padding: 4px 10px;
    border-radius: 12px;
    font-size: 0.75rem;
    cursor: pointer;
  }

  .diagram-card {
    background: #1E293B;
    border: 1px solid #334155;
    border-radius: 8px;
    padding: 12px;
  }

  .diagram-tag {
    font-size: 0.65rem;
    font-weight: 700;
    color: #38BDF8;
    margin-bottom: 6px;
  }

  .wf-door {
    display: flex;
    gap: 4px;
    text-align: center;
    font-size: 0.68rem;
    font-family: monospace;
    background: #0B1120;
    padding: 8px;
    border-radius: 4px;
  }

  .wf-ext, .wf-int { flex: 1; background: #1E293B; padding: 6px; border-radius: 2px; }
  .wf-edge { width: 80px; background: #334155; padding: 6px; border-radius: 2px; }

  .diagram-desc {
    font-size: 0.72rem;
    color: #94A3B8;
    margin: 6px 0 0 0;
  }

  .test-controls-group h3, .profile-data-card h3, .simulation-group h3 {
    margin: 0 0 8px 0;
    font-size: 0.85rem;
    color: #CBD5E1;
  }

  .btn-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 8px;
  }

  .sim-btn {
    background: #1E293B;
    border: 1px solid #475569;
    color: #F8FAFC;
    padding: 10px;
    border-radius: 6px;
    font-size: 0.75rem;
    font-weight: 600;
    cursor: pointer;
  }

  .sim-btn:hover { background: #334155; }
  .danger-sim { border-color: #EF4444; color: #FCA5A5; }
  .danger-sim:hover { background: #7F1D1D; }

  /* SECONDARY DEVICE (OPTION 2) */
  .secondary-device-card {
    background: #182234;
    border: 1px solid #334155;
    border-radius: 10px;
    padding: 12px;
    display: flex;
    justify-content: center;
  }

  .phone-frame {
    width: 220px;
    background: #020617;
    border: 3px solid #475569;
    border-radius: 20px;
    padding: 10px;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .phone-speaker {
    width: 40px;
    height: 4px;
    background: #334155;
    border-radius: 2px;
    margin-bottom: 8px;
  }

  .phone-screen {
    width: 100%;
    display: flex;
    flex-direction: column;
    gap: 8px;
  }

  .phone-time {
    font-size: 0.65rem;
    color: #64748B;
    text-align: center;
  }

  .phone-push-card {
    background: #1E293B;
    border-radius: 6px;
    padding: 6px 8px;
  }

  .push-app { font-size: 0.58rem; color: #38BDF8; font-weight: bold; }
  .phone-push-card p { margin: 2px 0 0 0; font-size: 0.68rem; color: #F1F5F9; }

  .phone-lock-toggle {
    width: 100%;
    padding: 6px;
    border-radius: 6px;
    border: none;
    font-size: 0.72rem;
    font-weight: bold;
    cursor: pointer;
  }

  .btn-primary { background: #3B82F6; color: white; }
  .btn-danger { background: #10B981; color: white; }

  /* HISTOGRAM & PROFILES (OPTION 3) */
  .profile-data-card {
    background: #182234;
    border: 1px solid #334155;
    border-radius: 10px;
    padding: 12px;
  }

  .profile-selector {
    display: flex;
    gap: 4px;
    flex-wrap: wrap;
    margin-bottom: 10px;
  }

  .profile-chip {
    background: #0B1120;
    border: 1px solid #334155;
    color: #94A3B8;
    font-size: 0.68rem;
    padding: 4px 8px;
    border-radius: 12px;
    cursor: pointer;
  }

  .active-profile {
    background: #4F46E5;
    color: white;
    border-color: #6366F1;
  }

  .histogram-box {
    background: #020617;
    border-radius: 6px;
    padding: 8px;
  }

  .chart-label {
    font-size: 0.65rem;
    color: #64748B;
    margin-bottom: 4px;
  }

  .bar-chart {
    width: 100%;
    height: 50px;
  }

  .chart-axis {
    display: flex;
    justify-content: space-between;
    font-size: 0.58rem;
    color: #475569;
  }

  /* SIMULATION RUNNER (OPTION 4) */
  .simulation-group {
    background: #182234;
    border: 1px solid #334155;
    border-radius: 10px;
    padding: 12px;
  }

  .sim-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 8px;
  }

  .sim-clock {
    font-size: 0.75rem;
    color: #38BDF8;
    font-family: monospace;
    font-weight: bold;
  }

  .sim-run-btn {
    width: 100%;
    background: #047857;
    color: white;
    border: none;
    padding: 10px;
    border-radius: 6px;
    font-weight: 700;
    font-size: 0.8rem;
    cursor: pointer;
  }

  .sim-active {
    background: #B91C1C !important;
  }

  /* MODALS */
  .modal-backdrop {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background: rgba(0, 0, 0, 0.75);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
  }

  .modal-window {
    background: #1E293B;
    border: 1px solid #475569;
    border-radius: 12px;
    padding: 24px;
    width: 90%;
    max-width: 440px;
  }

  .info-window {
    max-width: 560px;
  }

  .info-window ul {
    font-size: 0.8rem;
    color: #CBD5E1;
    line-height: 1.5;
  }

  .info-window code {
    background: #020617;
    color: #38BDF8;
    padding: 2px 4px;
    border-radius: 4px;
  }

  .form-group {
    margin-bottom: 12px;
    display: flex;
    flex-direction: column;
    gap: 4px;
  }

  .form-group label {
    font-size: 0.72rem;
    color: #94A3B8;
  }

  .form-group input, .form-group select {
    background: #0F172A;
    border: 1px solid #334155;
    color: white;
    padding: 8px;
    border-radius: 6px;
    font-size: 0.85rem;
  }

  .modal-buttons {
    display: flex;
    gap: 8px;
    margin-top: 16px;
  }

  .save-btn {
    flex: 1;
    background: #4F46E5;
    color: white;
    border: none;
    padding: 10px;
    border-radius: 6px;
    cursor: pointer;
    font-weight: bold;
  }

  .cancel-btn {
    background: transparent;
    color: #94A3B8;
    border: 1px solid #475569;
    padding: 10px;
    border-radius: 6px;
    cursor: pointer;
  }
</style>
