```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>LifePulse — Habits, Mood, Diary & Motivation</title>
  <!-- Chart.js & Firebase Compat SDKs -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.13.2/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.13.2/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.13.2/firebase-firestore-compat.js"></script>
  <style>
    :root {
      --bg: #f4f6fb;
      --surface: #ffffff;
      --surface-soft: #f8fafc;
      --text: #1e293b;
      --muted: #64748b;
      --primary: #4f46e5;
      --primary-hover: #4338ca;
      --primary-light: #eef2ff;
      --success: #10b981;
      --success-light: #d1fae5;
      --danger: #ef4444;
      --border: #e2e8f0;
      --shadow: 0 8px 24px rgba(15, 23, 42, 0.06);
      --radius: 14px;
      --habit-done-bg: #f0fdf4;
      --diary-text: #334155;
    }

    body.dark-mode {
      --bg: #0f172a;
      --surface: #1e293b;
      --surface-soft: #0f172a;
      --text: #f1f5f9;
      --muted: #94a3b8;
      --primary: #6366f1;
      --primary-hover: #818cf8;
      --primary-light: rgba(99, 102, 241, 0.18);
      --success: #10b981;
      --success-light: rgba(16, 185, 129, 0.18);
      --danger: #f87171;
      --border: #334155;
      --shadow: 0 10px 28px rgba(0, 0, 0, 0.35);
      --habit-done-bg: rgba(16, 185, 129, 0.12);
      --diary-text: #cbd5e1;
    }

    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
      background: var(--bg);
      color: var(--text);
      line-height: 1.5;
      min-height: 100vh;
      transition: background 0.25s ease, color 0.25s ease;
    }

    .hidden { display: none !important; }

    /* Animated Icons & Micro-Interactions */
    @keyframes iconBounce {
      0%, 100% { transform: translateY(0) scale(1); }
      40% { transform: translateY(-4px) scale(1.16); }
      70% { transform: translateY(1px) scale(0.95); }
    }
    @keyframes iconWiggle {
      0%, 100% { transform: rotate(0deg); }
      25% { transform: rotate(-10deg) scale(1.12); }
      75% { transform: rotate(10deg) scale(1.12); }
    }
    @keyframes pulseDot {
      0%, 100% { transform: scale(1); opacity: 1; }
      50% { transform: scale(1.25); opacity: 0.75; }
    }
    @keyframes popIn {
      0% { transform: scale(0.94); opacity: 0; }
      100% { transform: scale(1); opacity: 1; }
    }
    @keyframes fadeSlideUp {
      from { opacity: 0; transform: translateY(10px); }
      to { opacity: 1; transform: translateY(0); }
    }

    .anim-icon {
      display: inline-block;
      transition: transform 0.25s cubic-bezier(0.34, 1.56, 0.64, 1);
    }

    /* Ensure child elements inside interactive buttons never swallow taps */
    .drawer-item *,
    .hub-tile *,
    .hamburger-btn *,
    .back-btn *,
    .mood-option *,
    .theme-toggle-btn *,
    .profile-pill *,
    .icon-btn *,
    .btn-primary *,
    .btn-secondary * {
      pointer-events: none;
    }

    .drawer-item:hover .anim-icon,
    .hub-tile:hover .anim-icon,
    .icon-btn:hover .anim-icon,
    .btn-primary:hover .anim-icon,
    .btn-secondary:hover .anim-icon {
      animation: iconBounce 0.55s ease;
    }
    .drawer-item.active .anim-icon {
      transform: scale(1.18);
    }
    .mood-option:hover span {
      animation: iconWiggle 0.45s ease;
    }
    .mood-option.selected span {
      transform: scale(1.22);
    }

    button {
      cursor: pointer;
      font-family: inherit;
      border: none;
      transition: all 0.2s ease;
    }

    input, textarea, select {
      font-family: inherit;
      font-size: 0.95rem;
      padding: 0.7rem 0.9rem;
      border: 1px solid var(--border);
      border-radius: 10px;
      background: var(--surface);
      color: var(--text);
      outline: none;
      transition: border-color 0.2s, background 0.25s, color 0.25s;
    }

    input:focus, textarea:focus, select:focus {
      border-color: var(--primary);
      box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.16);
    }

    .btn-primary {
      background: var(--primary);
      color: #fff;
      padding: 0.7rem 1.2rem;
      border-radius: 10px;
      font-weight: 600;
    }
    .btn-primary:hover { background: var(--primary-hover); transform: translateY(-1px); }

    .btn-secondary {
      background: var(--primary-light);
      color: var(--primary);
      padding: 0.65rem 1rem;
      border-radius: 10px;
      font-weight: 600;
      display: inline-flex;
      align-items: center;
      justify-content: center;
      gap: 0.35rem;
      font-size: 0.9rem;
    }
    .btn-secondary.small { padding: 0.45rem 0.85rem; font-size: 0.85rem; }

    .btn-danger-outline {
      background: transparent;
      color: var(--danger);
      border: 1px solid rgba(239, 68, 68, 0.4);
      padding: 0.65rem 1rem;
      border-radius: 10px;
      font-weight: 600;
    }
    .btn-danger-outline:hover { background: rgba(239, 68, 68, 0.12); }

    .full-width { width: 100%; }

    .action-group {
      display: flex;
      align-items: center;
      gap: 0.25rem;
    }

    .icon-btn {
      background: transparent;
      color: var(--muted);
      font-size: 1.05rem;
      padding: 0.35rem 0.55rem;
      border-radius: 8px;
    }
    .icon-btn:hover { background: var(--surface-soft); color: var(--primary); }
    .icon-btn.danger:hover { color: var(--danger); }

    .theme-toggle-btn {
      background: var(--surface-soft);
      border: 1px solid var(--border);
      color: var(--text);
      width: 40px;
      height: 40px;
      border-radius: 11px;
      display: inline-flex;
      align-items: center;
      justify-content: center;
      font-size: 1.15rem;
      flex-shrink: 0;
    }
    .theme-toggle-btn:hover {
      border-color: var(--primary);
      transform: rotate(15deg);
    }

    /* Admin Announcement Banner */
    .admin-banner {
      background: linear-gradient(90deg, #4f46e5, #0ea5e9);
      color: #fff;
      padding: 0.65rem 1.1rem;
      border-radius: 12px;
      margin-bottom: 1rem;
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 0.75rem;
      font-size: 0.9rem;
      font-weight: 500;
      box-shadow: var(--shadow);
      animation: popIn 0.3s ease;
    }
    .admin-banner button {
      background: rgba(255, 255, 255, 0.2);
      color: #fff;
      border-radius: 6px;
      padding: 0.2rem 0.55rem;
      font-size: 0.82rem;
    }

    /* Auth Overlay */
    .auth-overlay {
      position: fixed;
      inset: 0;
      background: linear-gradient(135deg, #4f46e5 0%, #0ea5e9 100%);
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 1.5rem;
      z-index: 1300;
      overflow-y: auto;
    }

    .auth-card {
      background: var(--surface);
      width: 100%;
      max-width: 450px;
      padding: 2rem;
      border-radius: 20px;
      box-shadow: 0 25px 50px rgba(0, 0, 0, 0.22);
      text-align: center;
    }

    .brand-badge {
      display: inline-block;
      background: var(--primary-light);
      color: var(--primary);
      font-weight: 700;
      font-size: 0.85rem;
      padding: 0.3rem 0.8rem;
      border-radius: 999px;
      margin-bottom: 0.75rem;
    }

    .auth-card h1 { font-size: 1.6rem; margin-bottom: 0.4rem; }
    .subtitle { color: var(--muted); font-size: 0.92rem; margin-bottom: 1.3rem; }

    .btn-google {
      width: 100%;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 0.65rem;
      padding: 0.78rem 1rem;
      border-radius: 10px;
      background: var(--surface);
      border: 1px solid var(--border);
      font-weight: 600;
      font-size: 0.95rem;
      color: var(--text);
    }
    .btn-google:hover { background: var(--surface-soft); }

    .divider {
      display: flex;
      align-items: center;
      margin: 1.2rem 0;
      color: var(--muted);
      font-size: 0.82rem;
    }
    .divider::before, .divider::after {
      content: "";
      flex: 1;
      height: 1px;
      background: var(--border);
    }
    .divider span { padding: 0 0.75rem; }

    .auth-form {
      text-align: left;
      display: flex;
      flex-direction: column;
      gap: 0.9rem;
    }

    .avatar-upload-box {
      display: flex;
      align-items: center;
      gap: 1rem;
      justify-content: center;
      margin-bottom: 0.25rem;
    }
    .avatar-upload-box img {
      width: 70px;
      height: 70px;
      border-radius: 50%;
      object-fit: cover;
      border: 3px solid var(--primary-light);
    }

    .form-group { display: flex; flex-direction: column; gap: 0.3rem; }
    .form-group label { font-size: 0.85rem; font-weight: 600; color: var(--muted); }
    .hint-text { font-size: 0.82rem; color: var(--muted); margin-top: 0.45rem; }

    /* App Shell */
    .app-shell {
      max-width: 1120px;
      margin: 0 auto;
      padding: 1rem 1.25rem 3.5rem;
    }

    /* Home Topbar & Greeting */
    .topbar {
      display: flex;
      flex-wrap: wrap;
      justify-content: space-between;
      align-items: center;
      gap: 0.85rem;
      background: var(--surface);
      padding: 0.85rem 1.15rem;
      border-radius: var(--radius);
      box-shadow: var(--shadow);
      border: 1px solid var(--border);
      margin-bottom: 1rem;
    }

    .topbar-left {
      display: flex;
      align-items: center;
      gap: 0.85rem;
      flex-wrap: wrap;
    }

    .logo {
      display: flex;
      align-items: center;
      gap: 0.5rem;
      font-weight: 800;
      font-size: 1.2rem;
      color: var(--primary);
    }
    .logo-dot {
      width: 12px;
      height: 12px;
      border-radius: 50%;
      background: linear-gradient(135deg, var(--primary), var(--success));
      animation: pulseDot 2.2s infinite ease-in-out;
    }

    .live-clock {
      font-size: 0.85rem;
      font-weight: 500;
      color: var(--muted);
      background: var(--surface-soft);
      padding: 0.35rem 0.75rem;
      border-radius: 8px;
      border: 1px solid var(--border);
    }

    .topbar-right {
      display: flex;
      align-items: center;
      gap: 0.65rem;
      flex-wrap: wrap;
    }

    .sync-badge {
      font-size: 0.78rem;
      font-weight: 600;
      color: #059669;
      background: var(--success-light);
      padding: 0.3rem 0.7rem;
      border-radius: 999px;
    }

    .profile-pill {
      display: flex;
      align-items: center;
      gap: 0.6rem;
      background: var(--surface-soft);
      padding: 0.32rem 0.75rem 0.32rem 0.38rem;
      border-radius: 999px;
      border: 1px solid var(--border);
      color: var(--text);
    }
    .profile-pill:hover { border-color: var(--primary); }
    .profile-pill img { width: 34px; height: 34px; border-radius: 50%; object-fit: cover; }

    .profile-meta { display: flex; flex-direction: column; text-align: left; line-height: 1.2; }
    .profile-meta span { font-size: 0.86rem; font-weight: 600; }
    .profile-meta small { font-size: 0.72rem; color: var(--muted); }

    .greeting-hero {
      background: linear-gradient(135deg, rgba(79, 70, 229, 0.08), rgba(16, 185, 129, 0.08));
      border: 1px solid var(--border);
      border-radius: 16px;
      padding: 1.25rem 1.35rem;
      margin-bottom: 1.2rem;
    }
    .greeting-heading {
      font-size: 1.7rem;
      font-weight: 800;
      letter-spacing: -0.02em;
      color: var(--text);
      margin-bottom: 0.25rem;
    }
    .greeting-heading span {
      color: var(--primary);
    }
    .greeting-sub {
      color: var(--muted);
      font-size: 0.92rem;
    }

    /* Section Full-Page Sticky Topbar with Back Button */
    .section-topbar {
      position: sticky;
      top: 0;
      z-index: 40;
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 0.75rem;
      background: var(--surface);
      padding: 0.75rem 1rem;
      border-radius: var(--radius);
      box-shadow: var(--shadow);
      border: 1px solid var(--border);
      margin-bottom: 1.2rem;
    }

    .section-topbar-left {
      display: flex;
      align-items: center;
      gap: 0.6rem;
      min-width: 0;
    }

    .back-btn {
      background: var(--primary-light);
      color: var(--primary);
      border: 1px solid rgba(99, 102, 241, 0.25);
      padding: 0.55rem 0.95rem;
      border-radius: 10px;
      font-weight: 700;
      font-size: 0.92rem;
      display: inline-flex;
      align-items: center;
      gap: 0.35rem;
      flex-shrink: 0;
    }
    .back-btn:hover {
      background: var(--primary);
      color: #fff;
      transform: translateX(-2px);
    }

    .section-topbar-title {
      font-size: 1.08rem;
      font-weight: 800;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }

    /* Top-Left 3-Dash Hamburger Button & Slide-Out Drawer */
    .hamburger-btn {
      width: 42px;
      height: 42px;
      border-radius: 11px;
      background: var(--primary-light);
      color: var(--primary);
      border: 1px solid rgba(99, 102, 241, 0.25);
      display: inline-flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: 4px;
      padding: 0;
      flex-shrink: 0;
    }
    .hamburger-btn:hover {
      background: var(--primary);
      color: #fff;
      transform: scale(1.04);
    }
    .hamburger-btn span {
      display: block;
      width: 18px;
      height: 2.3px;
      background: currentColor;
      border-radius: 999px;
    }

    .drawer-backdrop {
      position: fixed;
      inset: 0;
      background: rgba(15, 23, 42, 0.55);
      backdrop-filter: blur(2px);
      z-index: 1090;
    }

    .nav-drawer {
      position: fixed;
      top: 0;
      left: 0;
      bottom: 0;
      width: min(300px, 84vw);
      background: var(--surface);
      border-right: 1px solid var(--border);
      box-shadow: 16px 0 40px rgba(15, 23, 42, 0.24);
      z-index: 1100;
      transform: translateX(-105%);
      transition: transform 0.28s cubic-bezier(0.22, 1, 0.36, 1);
      display: flex;
      flex-direction: column;
      padding: 1.1rem;
      overflow-y: auto;
    }
    .nav-drawer.open {
      transform: translateX(0);
    }

    .drawer-head {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding-bottom: 0.9rem;
      border-bottom: 1px solid var(--border);
      margin-bottom: 0.85rem;
    }

    .drawer-user-card {
      display: flex;
      align-items: center;
      gap: 0.7rem;
      background: var(--surface-soft);
      border: 1px solid var(--border);
      padding: 0.65rem 0.8rem;
      border-radius: 12px;
      margin-bottom: 0.9rem;
    }
    .drawer-user-card img {
      width: 38px;
      height: 38px;
      border-radius: 50%;
      object-fit: cover;
    }

    .drawer-nav-list {
      display: flex;
      flex-direction: column;
      gap: 0.38rem;
    }

    .drawer-item {
      width: 100%;
      background: transparent;
      color: var(--text);
      padding: 0.75rem 0.9rem;
      border-radius: 11px;
      font-weight: 600;
      font-size: 0.94rem;
      text-align: left;
      display: flex;
      align-items: center;
      gap: 0.75rem;
      border: 1px solid transparent;
    }
    .drawer-item .anim-icon {
      font-size: 1.25rem;
      width: 26px;
      text-align: center;
    }
    .drawer-item:hover {
      background: var(--primary-light);
      color: var(--primary);
      transform: translateX(4px);
    }
    .drawer-item.active {
      background: var(--primary);
      color: #fff;
      box-shadow: 0 4px 12px rgba(79, 70, 229, 0.25);
    }

    /* Home Hub Launcher Tiles */
    .hub-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
      gap: 0.9rem;
      margin-top: 0.85rem;
    }
    .hub-tile {
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: var(--radius);
      padding: 1.15rem 1rem;
      box-shadow: var(--shadow);
      display: flex;
      flex-direction: column;
      align-items: flex-start;
      gap: 0.45rem;
      text-align: left;
      color: var(--text);
    }
    .hub-tile:hover {
      border-color: var(--primary);
      transform: translateY(-3px);
    }
    .hub-tile-icon {
      font-size: 1.7rem;
      background: var(--primary-light);
      width: 46px;
      height: 46px;
      border-radius: 12px;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    .hub-tile strong {
      font-size: 0.98rem;
    }
    .hub-tile small {
      font-size: 0.78rem;
      color: var(--muted);
    }

    /* Tab Panels */
    .tab-panel {
      display: none;
      animation: fadeSlideUp 0.24s ease;
    }
    .tab-panel.active {
      display: block;
    }

    @media (max-width: 768px) {
      .app-shell {
        padding: 0.75rem 0.85rem 3rem;
      }
      .greeting-heading {
        font-size: 1.4rem;
      }
      .live-clock, .sync-badge {
        display: none;
      }
    }

    .panel-header {
      display: flex;
      justify-content: space-between;
      align-items: flex-end;
      flex-wrap: wrap;
      gap: 1rem;
      margin-bottom: 1.25rem;
    }
    .panel-header h2 { font-size: 1.45rem; margin-bottom: 0.2rem; }
    .panel-header p { color: var(--muted); font-size: 0.92rem; }

    .card {
      background: var(--surface);
      border-radius: var(--radius);
      padding: 1.25rem;
      box-shadow: var(--shadow);
      margin-bottom: 1.2rem;
      border: 1px solid var(--border);
    }

    /* Habits */
    .date-picker-wrap {
      display: flex;
      align-items: center;
      gap: 0.5rem;
      font-size: 0.9rem;
      font-weight: 600;
      color: var(--muted);
    }

    .progress-top {
      display: flex;
      justify-content: space-between;
      font-weight: 600;
      font-size: 0.92rem;
      margin-bottom: 0.6rem;
    }
    .progress-track {
      width: 100%;
      height: 12px;
      background: var(--surface-soft);
      border-radius: 999px;
      overflow: hidden;
      border: 1px solid var(--border);
    }
    .progress-fill {
      height: 100%;
      background: linear-gradient(90deg, var(--primary), var(--success));
      transition: width 0.35s ease;
    }

    .inline-form { display: flex; flex-wrap: wrap; gap: 0.75rem; }
    .inline-form input[type="text"] { flex: 1; min-width: 220px; }

    .habits-list { display: flex; flex-direction: column; gap: 0.75rem; }

    .habit-item {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 1rem;
      background: var(--surface);
      padding: 1rem 1.2rem;
      border-radius: var(--radius);
      box-shadow: var(--shadow);
      border: 1px solid var(--border);
      border-left: 4px solid var(--primary);
      transition: transform 0.2s ease, background 0.2s ease;
    }
    .habit-item:hover { transform: translateY(-1px); }
    .habit-item.completed { border-left-color: var(--success); background: var(--habit-done-bg); }

    .habit-left { display: flex; align-items: center; gap: 0.9rem; flex: 1; }
    .habit-checkbox { width: 24px; height: 24px; accent-color: var(--success); cursor: pointer; }
    .habit-info h4 { font-size: 1rem; margin-bottom: 0.15rem; }
    .habit-item.completed .habit-info h4 { text-decoration: line-through; color: var(--muted); }

    .habit-meta {
      display: flex;
      flex-wrap: wrap;
      gap: 0.5rem;
      font-size: 0.78rem;
      color: var(--muted);
    }

    .badge {
      background: var(--primary-light);
      color: var(--primary);
      padding: 0.15rem 0.55rem;
      border-radius: 999px;
      font-weight: 600;
      font-size: 0.75rem;
    }
    .badge.streak { background: #fef3c7; color: #b45309; }
    .badge.success { background: var(--success-light); color: #059669; }
    .badge.warn { background: #fef3c7; color: #b45309; }
    .badge.danger { background: rgba(239, 68, 68, 0.15); color: var(--danger); }

    /* Academics Section */
    .units-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
      gap: 1rem;
      margin-bottom: 1.25rem;
    }
    .unit-card {
      background: var(--surface);
      padding: 1.2rem;
      border-radius: var(--radius);
      box-shadow: var(--shadow);
      border: 1px solid var(--border);
      border-top: 4px solid var(--primary);
      display: flex;
      flex-direction: column;
      gap: 0.65rem;
      transition: transform 0.2s ease;
    }
    .unit-card:hover { transform: translateY(-2px); }
    .unit-card.on-track { border-top-color: var(--success); }
    .unit-card.needs-focus { border-top-color: #f59e0b; }
    .unit-card-top {
      display: flex;
      justify-content: space-between;
      align-items: flex-start;
      gap: 0.5rem;
    }
    .unit-metrics-row {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 0.45rem;
      background: var(--surface-soft);
      padding: 0.55rem;
      border-radius: 10px;
      border: 1px solid var(--border);
      text-align: center;
    }
    .unit-metric span { display: block; font-size: 0.7rem; color: var(--muted); font-weight: 600; }
    .unit-metric strong { font-size: 0.92rem; color: var(--text); }

    .weakness-list {
      display: flex;
      flex-direction: column;
      gap: 0.55rem;
      margin-top: 0.6rem;
    }
    .weakness-item {
      display: flex;
      justify-content: space-between;
      align-items: center;
      flex-wrap: wrap;
      gap: 0.5rem;
      padding: 0.7rem 0.9rem;
      border-radius: 10px;
      background: rgba(245, 158, 11, 0.1);
      border: 1px solid rgba(245, 158, 11, 0.3);
      font-size: 0.9rem;
    }

    .assessment-list {
      display: flex;
      flex-direction: column;
      gap: 0.75rem;
    }
    .assessment-item {
      display: flex;
      justify-content: space-between;
      align-items: center;
      flex-wrap: wrap;
      gap: 0.85rem;
      background: var(--surface);
      padding: 1rem 1.15rem;
      border-radius: var(--radius);
      box-shadow: var(--shadow);
      border: 1px solid var(--border);
    }
    .score-pill {
      display: inline-flex;
      align-items: center;
      gap: 0.4rem;
      padding: 0.35rem 0.75rem;
      border-radius: 10px;
      font-weight: 700;
      font-size: 0.9rem;
      background: var(--primary-light);
      color: var(--primary);
    }
    .score-pill.high { background: var(--success-light); color: #059669; }
    .score-pill.mid { background: #fef3c7; color: #b45309; }
    .score-pill.low { background: rgba(239, 68, 68, 0.15); color: var(--danger); }

    .cal-dot.academic { background: #8b5cf6; }

    /* Calendar Section */
    .calendar-layout {
      display: grid;
      grid-template-columns: 1.2fr 1fr;
      gap: 1.2rem;
      align-items: start;
    }
    @media (max-width: 860px) {
      .calendar-layout { grid-template-columns: 1fr; }
    }
    .cal-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 1rem;
      gap: 0.5rem;
    }
    .cal-header h3 { font-size: 1.15rem; }
    .cal-nav-btns { display: flex; gap: 0.4rem; }
    .cal-weekdays {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      text-align: center;
      font-size: 0.78rem;
      font-weight: 700;
      color: var(--muted);
      margin-bottom: 0.45rem;
    }
    .cal-grid {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 0.35rem;
    }
    .cal-day {
      aspect-ratio: 1 / 1;
      min-height: 46px;
      border-radius: 11px;
      background: var(--surface-soft);
      border: 1px solid var(--border);
      color: var(--text);
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: 0.2rem;
      padding: 0.25rem;
      position: relative;
      font-weight: 600;
      font-size: 0.88rem;
    }
    .cal-day:hover {
      border-color: var(--primary);
      transform: scale(1.04);
    }
    .cal-day.empty {
      background: transparent;
      border-color: transparent;
      pointer-events: none;
    }
    .cal-day.today {
      border: 2px solid var(--success);
    }
    .cal-day.selected {
      background: var(--primary);
      color: #fff;
      border-color: var(--primary);
      box-shadow: 0 6px 14px rgba(79, 70, 229, 0.3);
    }
    .cal-dots {
      display: flex;
      gap: 3px;
      align-items: center;
      min-height: 8px;
    }
    .cal-dot {
      width: 6px;
      height: 6px;
      border-radius: 50%;
    }
    .cal-dot.habit { background: var(--success); }
    .cal-dot.mood { background: #f59e0b; }
    .cal-dot.diary { background: #0ea5e9; }
    .cal-day.selected .cal-dot { background: #fff; }

    .cal-legend {
      display: flex;
      flex-wrap: wrap;
      gap: 0.9rem;
      margin-top: 0.9rem;
      font-size: 0.78rem;
      color: var(--muted);
    }
    .cal-legend span { display: inline-flex; align-items: center; gap: 0.35rem; }

    .inspector-section {
      margin-top: 0.9rem;
      padding-top: 0.85rem;
      border-top: 1px solid var(--border);
    }
    .inspector-section h4 {
      font-size: 0.88rem;
      color: var(--muted);
      margin-bottom: 0.5rem;
      text-transform: uppercase;
      letter-spacing: 0.04em;
    }
    .inspector-item {
      background: var(--surface-soft);
      padding: 0.65rem 0.85rem;
      border-radius: 10px;
      margin-bottom: 0.45rem;
      font-size: 0.9rem;
      border: 1px solid var(--border);
    }

    /* Analytics */
    .stats-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(155px, 1fr));
      gap: 0.9rem;
      margin-bottom: 1.2rem;
    }
    .stat-card { margin-bottom: 0; display: flex; flex-direction: column; gap: 0.35rem; }
    .stat-label { font-size: 0.85rem; color: var(--muted); font-weight: 600; }
    .stat-value { font-size: 1.65rem; color: var(--primary); }

    .charts-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(290px, 1fr));
      gap: 1rem;
      margin-bottom: 1.2rem;
    }
    .chart-card h3 { font-size: 1rem; margin-bottom: 0.8rem; }
    .chart-wrap { position: relative; height: 250px; }

    .activity-list {
      list-style: none;
      margin-top: 0.75rem;
      display: flex;
      flex-direction: column;
      gap: 0.6rem;
    }
    .activity-list li {
      display: flex;
      justify-content: space-between;
      flex-wrap: wrap;
      gap: 0.5rem;
      font-size: 0.9rem;
      padding: 0.6rem 0.8rem;
      background: var(--surface-soft);
      border-radius: 8px;
    }

    /* Mood */
    .mood-creator h3 { font-size: 1rem; margin-bottom: 0.75rem; }
    .mood-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(105px, 1fr));
      gap: 0.65rem;
      margin-bottom: 1.25rem;
    }

    .mood-option {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 0.3rem;
      padding: 0.75rem 0.5rem;
      background: var(--surface-soft);
      border: 2px solid transparent;
      border-radius: 12px;
      font-weight: 600;
      font-size: 0.85rem;
      color: var(--text);
    }
    .mood-option span { font-size: 1.6rem; transition: transform 0.2s ease; }
    .mood-option:hover { background: var(--primary-light); }
    .mood-option.selected {
      border-color: var(--primary);
      background: var(--primary-light);
      color: var(--primary);
    }

    .mood-creator textarea { width: 100%; margin-bottom: 0.8rem; }
    .form-actions {
      display: flex;
      justify-content: space-between;
      align-items: center;
      flex-wrap: wrap;
      gap: 0.75rem;
    }

    .entries-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
      gap: 1rem;
      margin-top: 0.8rem;
    }

    .entry-card {
      background: var(--surface);
      padding: 1.1rem;
      border-radius: var(--radius);
      box-shadow: var(--shadow);
      border: 1px solid var(--border);
      display: flex;
      flex-direction: column;
      gap: 0.5rem;
    }
    .entry-top { display: flex; justify-content: space-between; align-items: center; }
    .entry-mood-title { font-weight: 700; font-size: 1.05rem; }
    .entry-time { font-size: 0.78rem; color: var(--muted); }

    /* Diary */
    .stack-form { display: flex; flex-direction: column; gap: 0.85rem; }
    .stack-form h3 { font-size: 1.05rem; }
    .form-row { display: flex; flex-wrap: wrap; gap: 0.75rem; }
    .form-row input { flex: 1; min-width: 200px; }

    .search-bar-wrap { margin-bottom: 1rem; }
    .search-bar-wrap input { width: 100%; }

    .diary-list { display: flex; flex-direction: column; gap: 1rem; }
    .diary-card {
      background: var(--surface);
      padding: 1.25rem;
      border-radius: var(--radius);
      box-shadow: var(--shadow);
      border: 1px solid var(--border);
    }
    .diary-card-head {
      display: flex;
      justify-content: space-between;
      align-items: flex-start;
      gap: 1rem;
      margin-bottom: 0.6rem;
    }
    .diary-card h4 { font-size: 1.12rem; }
    .diary-body {
      white-space: pre-wrap;
      color: var(--diary-text);
      font-size: 0.95rem;
      margin-top: 0.5rem;
    }

    /* Advice */
    .spotlight-card {
      background: linear-gradient(135deg, #4f46e5, #6366f1);
      color: #fff;
      border: none;
    }
    .spotlight-label {
      font-size: 0.78rem;
      text-transform: uppercase;
      letter-spacing: 0.08em;
      opacity: 0.85;
      font-weight: 700;
    }
    .spotlight-card blockquote {
      font-size: 1.25rem;
      font-weight: 600;
      margin: 0.75rem 0;
      line-height: 1.45;
    }
    .spotlight-footer {
      display: flex;
      justify-content: space-between;
      align-items: center;
      flex-wrap: wrap;
      gap: 0.75rem;
    }
    .spotlight-footer cite { font-style: normal; opacity: 0.92; font-weight: 500; }

    .filter-bar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      flex-wrap: wrap;
      gap: 0.75rem;
      margin: 1rem 0;
    }

    .advice-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(290px, 1fr));
      gap: 1rem;
    }
    .advice-card {
      background: var(--surface);
      padding: 1.2rem;
      border-radius: var(--radius);
      box-shadow: var(--shadow);
      border: 1px solid var(--border);
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      gap: 0.8rem;
    }
    .advice-quote { font-size: 0.98rem; font-weight: 500; color: var(--text); }
    .advice-footer {
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-size: 0.82rem;
      color: var(--muted);
    }

    .like-btn {
      background: var(--surface-soft);
      padding: 0.3rem 0.65rem;
      border-radius: 999px;
      font-size: 0.82rem;
      font-weight: 600;
      color: var(--muted);
    }
    .like-btn.liked { background: #fee2e2; color: #dc2626; }

    /* Admin Section */
    .admin-lock-box {
      max-width: 420px;
      margin: 2rem auto;
      text-align: center;
    }
    .admin-quote-row {
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 0.75rem;
      padding: 0.75rem;
      background: var(--surface-soft);
      border: 1px solid var(--border);
      border-radius: 10px;
      margin-bottom: 0.5rem;
    }

    /* Modal & Toast */
    .modal-backdrop {
      position: fixed;
      inset: 0;
      background: rgba(15, 23, 42, 0.6);
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 1rem;
      z-index: 1200;
      overflow-y: auto;
    }
    .modal-card {
      background: var(--surface);
      width: 100%;
      max-width: 480px;
      border-radius: 18px;
      padding: 1.5rem;
      box-shadow: 0 25px 50px rgba(0, 0, 0, 0.25);
      border: 1px solid var(--border);
      max-height: 92vh;
      overflow-y: auto;
      animation: popIn 0.2s ease;
    }
    .modal-head {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 1rem;
    }
    .modal-actions {
      display: flex;
      justify-content: space-between;
      flex-wrap: wrap;
      gap: 0.75rem;
      margin-top: 0.5rem;
    }
    .modal-divider {
      border: none;
      border-top: 1px solid var(--border);
      margin: 1.1rem 0;
    }

    .toast {
      position: fixed;
      bottom: 1.5rem;
      right: 1.5rem;
      background: #1e293b;
      color: #fff;
      padding: 0.75rem 1.2rem;
      border-radius: 10px;
      font-size: 0.9rem;
      font-weight: 500;
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.25);
      z-index: 1500;
    }

    .empty-state {
      text-align: center;
      color: var(--muted);
      padding: 2rem 1rem;
      background: var(--surface);
      border-radius: var(--radius);
      border: 1px dashed var(--border);
    }
  </style>
</head>
<body>

  <!-- ACCOUNT CREATION / GOOGLE SIGN-IN OVERLAY -->
  <div id="authOverlay" class="auth-overlay hidden">
    <div class="auth-card">
      <div class="brand-badge">LifePulse</div>
      <h1>Create Your Space</h1>
      <p class="subtitle">Upload a profile picture, connect your Google account, and track your life in real time.</p>

      <button id="googleSignInBtn" class="btn-google" type="button">
        <svg viewBox="0 0 24 24" width="20" height="20" aria-hidden="true">
          <path fill="#4285F4" d="M23.745 12.27c0-.7-.06-1.4-.19-2.07H12v4.51h6.6c-.29 1.52-1.14 2.82-2.4 3.68v3.05h3.88c2.27-2.09 3.665-5.17 3.665-9.17z"/>
          <path fill="#34A853" d="M12 24c3.24 0 5.95-1.08 7.93-2.91l-3.88-3.05c-1.08.72-2.45 1.16-4.05 1.16-3.12 0-5.77-2.11-6.72-4.96H1.29v3.14C3.26 21.3 7.31 24 12 24z"/>
          <path fill="#FBBC05" d="M5.28 14.24c-.24-.72-.38-1.49-.38-2.24s.14-1.52.38-2.24V6.62H1.29C.47 8.24 0 10.06 0 12s.47 3.76 1.29 5.38l3.99-3.14z"/>
          <path fill="#EA4335" d="M12 4.75c1.77 0 3.35.61 4.6 1.8l3.42-3.42C17.95 1.19 15.24 0 12 0 7.31 0 3.26 2.7 1.29 6.62l3.99 3.14c.95-2.85 3.6-4.96 6.72-4.96z"/>
        </svg>
        Continue with Google
      </button>

      <div class="divider"><span>or create your Google profile &amp; photo here</span></div>

      <form id="localAccountForm" class="auth-form">
        <div class="avatar-upload-box">
          <img id="authAvatarPreview" src="https://ui-avatars.com/api/?name=User&background=4f46e5&color=fff" alt="Profile Preview" />
          <label for="authAvatarInput" class="btn-secondary small">Upload Profile Pic</label>
          <input type="file" id="authAvatarInput" accept="image/*" hidden />
        </div>

        <div class="form-group">
          <label for="authName">Full Name</label>
          <input type="text" id="authName" placeholder="e.g., Alex Kamau" required />
        </div>

        <div class="form-group">
          <label for="authEmail">Google Email (@gmail.com)</label>
          <input type="email" id="authEmail" placeholder="e.g., alex@gmail.com" required />
        </div>

        <button type="submit" class="btn-primary full-width">Create Account &amp; Start</button>
      </form>
      <p id="firebaseStatusNote" class="hint-text"></p>
    </div>
  </div>

  <!-- SLIDE-OUT 3-DASH HAMBURGER DRAWER -->
  <div id="drawerBackdrop" class="drawer-backdrop hidden"></div>
  <aside id="navDrawer" class="nav-drawer" aria-label="Main Sections">
    <div class="drawer-head">
      <div class="logo">
        <span class="logo-dot"></span>
        <span>LifePulse Menu</span>
      </div>
      <button id="closeDrawerBtn" type="button" class="icon-btn" title="Close Menu">✕</button>
    </div>

    <div class="drawer-user-card">
      <img id="drawerAvatar" src="https://ui-avatars.com/api/?name=User&background=4f46e5&color=fff" alt="User" />
      <div class="profile-meta">
        <span id="drawerUserName">Guest User</span>
        <small>Tap any icon to open</small>
      </div>
    </div>

    <nav class="drawer-nav-list">
      <button type="button" class="drawer-item active" data-nav-target="home"><span class="anim-icon">🏠</span><span>Home Overview</span></button>
      <button type="button" class="drawer-item" data-nav-target="habits"><span class="anim-icon">✅</span><span>Habits</span></button>
      <button type="button" class="drawer-item" data-nav-target="academics"><span class="anim-icon">🎓</span><span>Academics</span></button>
      <button type="button" class="drawer-item" data-nav-target="calendar"><span class="anim-icon">📅</span><span>Calendar</span></button>
      <button type="button" class="drawer-item" data-nav-target="analytics"><span class="anim-icon">📊</span><span>Analytics</span></button>
      <button type="button" class="drawer-item" data-nav-target="mood"><span class="anim-icon">😊</span><span>Mood</span></button>
      <button type="button" class="drawer-item" data-nav-target="diary"><span class="anim-icon">📔</span><span>Diary</span></button>
      <button type="button" class="drawer-item" data-nav-target="advice"><span class="anim-icon">💡</span><span>Advice &amp; Quotes</span></button>
      <button type="button" class="drawer-item" data-nav-target="admin"><span class="anim-icon">🛡️</span><span>Admin</span></button>
    </nav>
  </aside>

  <!-- MAIN APP CONTAINER -->
  <div id="appContainer" class="app-shell">
    <!-- Admin Announcement Banner -->
    <div id="adminAnnouncementBar" class="admin-banner hidden">
      <span id="adminAnnouncementText"></span>
      <button type="button" id="dismissBannerBtn" title="Hide banner">✕</button>
    </div>

    <!-- HOME HEADER (Visible on Home Overview) -->
    <header id="homeHeader">
      <div class="topbar">
        <div class="topbar-left">
          <button id="menuToggleBtn" class="hamburger-btn" type="button" title="Open Menu" aria-label="Open Menu">
            <span></span>
            <span></span>
            <span></span>
          </button>
          <div class="logo">
            <span class="logo-dot"></span>
            <span>LifePulse</span>
          </div>
          <div class="live-clock" id="liveClock">Loading clock...</div>
        </div>

        <div class="topbar-right">
          <span id="syncBadge" class="sync-badge">● Saved Real-Time</span>
          <button id="darkModeToggle" class="theme-toggle-btn" type="button" title="Toggle Dark / Light Mode" aria-label="Toggle Theme">
            <span id="themeToggleIcon" class="anim-icon">🌙</span>
          </button>
          <button id="openProfileBtn" class="profile-pill" type="button" title="Profile, Picture &amp; Backup Settings">
            <img id="headerAvatar" src="https://ui-avatars.com/api/?name=User&background=4f46e5&color=fff" alt="Avatar" />
            <div class="profile-meta">
              <span id="headerName">Guest User</span>
              <small id="headerEmail">Tap to edit profile</small>
            </div>
          </button>
        </div>
      </div>

      <div class="greeting-hero">
        <h1 class="greeting-heading">Hello, <span id="greetingUserName">Guest</span>! 👋</h1>
        <p class="greeting-sub">Tap the <strong>☰</strong> 3-dash button on the top left or choose any section below to open it full screen.</p>
      </div>
    </header>

    <!-- FULL-PAGE SECTION TOPBAR WITH BACK BUTTON (Visible when inside any section) -->
    <header id="sectionTopbar" class="section-topbar hidden">
      <div class="section-topbar-left">
        <button id="sectionBackBtn" type="button" class="back-btn" title="Back to Home">← Back</button>
        <button id="sectionMenuBtn" class="hamburger-btn" type="button" title="Open Menu" aria-label="Open Menu">
          <span></span>
          <span></span>
          <span></span>
        </button>
        <span id="sectionTopbarTitle" class="section-topbar-title">Section</span>
      </div>

      <div class="topbar-right">
        <button id="sectionDarkModeToggle" class="theme-toggle-btn" type="button" title="Toggle Dark / Light Mode" aria-label="Toggle Theme">
          <span id="sectionThemeIcon" class="anim-icon">🌙</span>
        </button>
        <button id="sectionProfileBtn" class="profile-pill" type="button" title="Profile Settings">
          <img id="sectionAvatar" src="https://ui-avatars.com/api/?name=User&background=4f46e5&color=fff" alt="Avatar" />
        </button>
      </div>
    </header>

    <main class="main-content">
      <!-- HOME OVERVIEW -->
      <section id="tab-home" class="tab-panel active">
        <div class="stats-grid">
          <div class="card stat-card">
            <span class="stat-label">Today's Habits</span>
            <strong id="hubHabitSummary" class="stat-value">0 / 0</strong>
          </div>
          <div class="card stat-card">
            <span class="stat-label">Academic Mean</span>
            <strong id="hubAcademicSummary" class="stat-value">0%</strong>
          </div>
          <div class="card stat-card">
            <span class="stat-label">Moods Logged</span>
            <strong id="hubMoodSummary" class="stat-value">0</strong>
          </div>
          <div class="card stat-card">
            <span class="stat-label">Diary Entries</span>
            <strong id="hubDiarySummary" class="stat-value">0</strong>
          </div>
        </div>

        <div class="card">
          <h3 style="margin-bottom:0.2rem;">Your Sections</h3>
          <p class="hint-text" style="margin-top:0;">Tap any animated icon tile to dive into that full workspace.</p>
          <div class="hub-grid">
            <button type="button" class="hub-tile" data-nav-target="habits">
              <span class="hub-tile-icon"><span class="anim-icon">✅</span></span>
              <strong>Habits</strong>
              <small>Daily goals &amp; streaks</small>
            </button>
            <button type="button" class="hub-tile" data-nav-target="academics">
              <span class="hub-tile-icon"><span class="anim-icon">🎓</span></span>
              <strong>Academics</strong>
              <small>Units, CATs &amp; revision</small>
            </button>
            <button type="button" class="hub-tile" data-nav-target="calendar">
              <span class="hub-tile-icon"><span class="anim-icon">📅</span></span>
              <strong>Calendar</strong>
              <small>Inspect any date</small>
            </button>
            <button type="button" class="hub-tile" data-nav-target="analytics">
              <span class="hub-tile-icon"><span class="anim-icon">📊</span></span>
              <strong>Analytics</strong>
              <small>Charts &amp; progress</small>
            </button>
            <button type="button" class="hub-tile" data-nav-target="mood">
              <span class="hub-tile-icon"><span class="anim-icon">😊</span></span>
              <strong>Mood</strong>
              <small>Log how you feel</small>
            </button>
            <button type="button" class="hub-tile" data-nav-target="diary">
              <span class="hub-tile-icon"><span class="anim-icon">📔</span></span>
              <strong>Diary</strong>
              <small>Personal journal</small>
            </button>
            <button type="button" class="hub-tile" data-nav-target="advice">
              <span class="hub-tile-icon"><span class="anim-icon">💡</span></span>
              <strong>Quotes</strong>
              <small>Advice &amp; motivation</small>
            </button>
            <button type="button" class="hub-tile" data-nav-target="admin">
              <span class="hub-tile-icon"><span class="anim-icon">🛡️</span></span>
              <strong>Admin</strong>
              <small>Owner control center</small>
            </button>
          </div>
        </div>
      </section>

      <!-- HABITS -->
      <section id="tab-habits" class="tab-panel">
        <div class="panel-header">
          <div>
            <h2>✅ Habit Tracker</h2>
            <p>Write a habit you are trying to achieve and tick it off every day you complete it.</p>
          </div>
          <div class="date-picker-wrap">
            <label for="habitDatePicker">Date:</label>
            <input type="date" id="habitDatePicker" />
          </div>
        </div>

        <div class="card progress-card">
          <div class="progress-top">
            <span id="todayProgressText">0 of 0 habits completed</span>
            <span id="todayProgressPct">0%</span>
          </div>
          <div class="progress-track">
            <div id="todayProgressBar" class="progress-fill" style="width: 0%;"></div>
          </div>
        </div>

        <form id="habitForm" class="card inline-form">
          <input type="text" id="habitTitle" placeholder="Write a habit to achieve (e.g., Drink 2L water, Study 1 hour)..." required />
          <select id="habitCategory">
            <option value="Health">Health</option>
            <option value="Productivity">Productivity</option>
            <option value="Mindfulness">Mindfulness</option>
            <option value="Learning">Learning</option>
            <option value="Fitness">Fitness</option>
          </select>
          <button type="submit" class="btn-primary"><span class="anim-icon">➕</span> Add Habit</button>
        </form>

        <div id="habitsList" class="habits-list"></div>
      </section>

      <!-- ACADEMICS -->
      <section id="tab-academics" class="tab-panel">
        <div class="panel-header">
          <div>
            <h2>🎓 Academic Progress &amp; Revision Tracker</h2>
            <p>Manage your course units, record CATs, assignments &amp; topical revision scores, and spot weak topics early.</p>
          </div>
        </div>

        <!-- Academic KPI Summary -->
        <div class="stats-grid">
          <div class="card stat-card">
            <span class="stat-label">Overall Mean</span>
            <strong id="acadStatMean" class="stat-value">0%</strong>
          </div>
          <div class="card stat-card">
            <span class="stat-label">Assessments Logged</span>
            <strong id="acadStatTotalCount" class="stat-value">0</strong>
          </div>
          <div class="card stat-card">
            <span class="stat-label">Strongest Unit</span>
            <strong id="acadStatBestUnit" class="stat-value" style="font-size:1.15rem;">—</strong>
          </div>
          <div class="card stat-card">
            <span class="stat-label">Units On Target</span>
            <strong id="acadStatOnTarget" class="stat-value">0 / 0</strong>
          </div>
        </div>

        <!-- Step 1: Add Course Unit -->
        <form id="unitForm" class="card stack-form">
          <h3>1. Add a Course Unit</h3>
          <div class="form-row">
            <input type="text" id="unitCode" placeholder="Unit Code (e.g., EME 2101)" required style="max-width:200px;" />
            <input type="text" id="unitName" placeholder="Unit Name (e.g., Engineering Mechanics)" required />
          </div>
          <div class="form-row">
            <input type="text" id="unitSemester" placeholder="Semester / Term (e.g., Y2 Sem 1)" value="Sem 1" />
            <input type="number" id="unitTarget" min="1" max="100" value="70" placeholder="Target %" style="max-width:160px;" />
            <input type="text" id="unitNotes" placeholder="Lecturer or focus notes (optional)" />
            <button type="submit" class="btn-primary"><span class="anim-icon">📘</span> Add Unit</button>
          </div>
        </form>

        <!-- Step 2: Record CAT, Assignment, or Topical Revision Score -->
        <form id="assessmentForm" class="card stack-form">
          <h3>2. Record CAT Marks, Assignment, or Topical Revision Result</h3>
          <div class="form-row">
            <select id="assessUnitSelect" required>
              <option value="">-- Select Course Unit --</option>
            </select>
            <select id="assessType">
              <option value="CAT 1">CAT 1</option>
              <option value="CAT 2">CAT 2</option>
              <option value="Assignment">Assignment</option>
              <option value="Topical Revision">Topical Revision</option>
              <option value="Lab / Practical">Lab / Practical</option>
            </select>
            <input type="text" id="assessTopic" placeholder="Topic / Title (e.g., Bending Moments, Integration)" required />
          </div>
          <div class="form-row">
            <input type="number" id="assessScore" step="0.1" min="0" placeholder="Marks Scored (e.g., 24)" required style="max-width:210px;" />
            <input type="number" id="assessTotal" step="0.1" min="1" placeholder="Out of Total (e.g., 30)" required style="max-width:210px;" />
            <input type="date" id="assessDate" />
            <input type="text" id="assessNote" placeholder="Key takeaway or mistakes to revise..." />
            <button type="submit" class="btn-primary"><span class="anim-icon">📝</span> Save Result</button>
          </div>
        </form>

        <!-- Search & Filter Bar -->
        <div class="card" style="padding:1rem 1.2rem;">
          <div class="form-row" style="align-items:center;">
            <input type="search" id="acadSearchInput" placeholder="🔍 Search units, topics, CATs, assignments, or revision notes..." />
            <select id="acadFilterUnit">
              <option value="All">All Units</option>
            </select>
            <select id="acadFilterType">
              <option value="All">All Types</option>
              <option value="CAT">CATs (CAT 1 &amp; 2)</option>
              <option value="Assignment">Assignments</option>
              <option value="Topical Revision">Topical Revision</option>
              <option value="Lab / Practical">Lab / Practical</option>
            </select>
          </div>
        </div>

        <!-- Units Progress Cards -->
        <h3 style="margin-bottom:0.75rem;">📚 Enrolled Units &amp; Progress</h3>
        <div id="unitsGrid" class="units-grid"></div>

        <!-- Academic Analytics Charts -->
        <div class="charts-grid">
          <div class="card chart-card">
            <h3>Unit Average vs. Target (%)</h3>
            <div class="chart-wrap"><canvas id="unitComparisonChart"></canvas></div>
          </div>
          <div class="card chart-card">
            <h3>Assessment &amp; Revision Score Trend (%)</h3>
            <div class="chart-wrap"><canvas id="academicTrendChart"></canvas></div>
          </div>
        </div>

        <!-- Weak Topics & Revision Priority Analyzer -->
        <div class="card">
          <h3>🎯 Weak Topics &amp; Revision Priority Analyzer</h3>
          <p class="hint-text" style="margin-top:0;">Automatically flags topics scoring below 60% or trailing your unit target so you know what to revise next.</p>
          <div id="academicWeaknessList" class="weakness-list"></div>
        </div>

        <!-- Assessment & Revision History -->
        <h3 style="margin:1.1rem 0 0.75rem;">🗂️ Recorded CATs, Assignments &amp; Revision Results</h3>
        <div id="assessmentsHistoryList" class="assessment-list"></div>
      </section>

      <!-- CALENDAR -->
      <section id="tab-calendar" class="tab-panel">
        <div class="panel-header">
          <div>
            <h2>📅 Life Calendar</h2>
            <p>Tap any date to inspect the habits completed, assessments logged, moods, and diary entries written that day.</p>
          </div>
        </div>

        <div class="calendar-layout">
          <div class="card">
            <div class="cal-header">
              <h3 id="calMonthLabel">Month Year</h3>
              <div class="cal-nav-btns">
                <button type="button" id="calPrevBtn" class="btn-secondary small">◀ Prev</button>
                <button type="button" id="calTodayBtn" class="btn-secondary small">Today</button>
                <button type="button" id="calNextBtn" class="btn-secondary small">Next ▶</button>
              </div>
            </div>

            <div class="cal-weekdays">
              <span>Sun</span><span>Mon</span><span>Tue</span><span>Wed</span><span>Thu</span><span>Fri</span><span>Sat</span>
            </div>
            <div id="calendarGrid" class="cal-grid"></div>

            <div class="cal-legend">
              <span><i class="cal-dot habit"></i> Habit</span>
              <span><i class="cal-dot academic"></i> Academic</span>
              <span><i class="cal-dot mood"></i> Mood</span>
              <span><i class="cal-dot diary"></i> Diary</span>
            </div>
          </div>

          <div class="card" id="dayInspectorCard">
            <div style="display:flex; justify-content:space-between; align-items:center; flex-wrap:wrap; gap:0.5rem;">
              <h3 id="inspectorDateTitle">Selected Date</h3>
              <div style="display:flex; gap:0.4rem;">
                <button type="button" id="quickLogMoodDateBtn" class="btn-secondary small">+ Mood</button>
                <button type="button" id="quickLogDiaryDateBtn" class="btn-secondary small">+ Diary</button>
              </div>
            </div>
            <div id="dayInspectorContent"></div>
          </div>
        </div>
      </section>

      <!-- ANALYTICS -->
      <section id="tab-analytics" class="tab-panel">
        <div class="panel-header">
          <div>
            <h2>📊 Habit &amp; Mood Analytics</h2>
            <p>Real-time insights into your completion rate, streaks, mood distribution, and check-in timestamps.</p>
          </div>
        </div>

        <div class="stats-grid">
          <div class="card stat-card">
            <span class="stat-label">Today's Rate</span>
            <strong id="statTodayRate" class="stat-value">0%</strong>
          </div>
          <div class="card stat-card">
            <span class="stat-label">Best Streak</span>
            <strong id="statStreak" class="stat-value">0 days</strong>
          </div>
          <div class="card stat-card">
            <span class="stat-label">Total Check-ins</span>
            <strong id="statTotalCheckins" class="stat-value">0</strong>
          </div>
          <div class="card stat-card">
            <span class="stat-label">Moods Logged</span>
            <strong id="statTotalMoods" class="stat-value">0</strong>
          </div>
        </div>

        <div class="charts-grid">
          <div class="card chart-card">
            <h3>Last 7 Days Habit Completion (%)</h3>
            <div class="chart-wrap"><canvas id="weeklyChart"></canvas></div>
          </div>
          <div class="card chart-card">
            <h3>Total Check-ins by Habit</h3>
            <div class="chart-wrap"><canvas id="habitBreakdownChart"></canvas></div>
          </div>
          <div class="card chart-card">
            <h3>Mood Frequency &amp; Trends</h3>
            <div class="chart-wrap"><canvas id="moodTrendChart"></canvas></div>
          </div>
        </div>

        <div class="card">
          <h3>Real-Time Activity Log</h3>
          <ul id="recentActivityList" class="activity-list"></ul>
        </div>
      </section>

      <!-- MOOD -->
      <section id="tab-mood" class="tab-panel">
        <div class="panel-header">
          <div>
            <h2>😊 Mood Tracker</h2>
            <p>Choose your mood from the choices below, write what your mood is, and save it.</p>
          </div>
        </div>

        <div class="card mood-creator">
          <h3>1. Choose your mood</h3>
          <div class="mood-grid">
            <button type="button" class="mood-option" data-mood="Happy" data-emoji="😄"><span>😄</span>Happy</button>
            <button type="button" class="mood-option" data-mood="Calm" data-emoji="😌"><span>😌</span>Calm</button>
            <button type="button" class="mood-option" data-mood="Motivated" data-emoji="🔥"><span>🔥</span>Motivated</button>
            <button type="button" class="mood-option" data-mood="Grateful" data-emoji="🙏"><span>🙏</span>Grateful</button>
            <button type="button" class="mood-option" data-mood="Tired" data-emoji="😴"><span>😴</span>Tired</button>
            <button type="button" class="mood-option" data-mood="Stressed" data-emoji="😰"><span>😰</span>Stressed</button>
            <button type="button" class="mood-option" data-mood="Sad" data-emoji="😢"><span>😢</span>Sad</button>
            <button type="button" class="mood-option" data-mood="Anxious" data-emoji="😟"><span>😟</span>Anxious</button>
          </div>

          <h3>2. Write what your mood is</h3>
          <textarea id="moodNote" rows="3" placeholder="Write why you feel this way or what's on your mind..."></textarea>

          <div class="form-actions">
            <span id="selectedMoodIndicator" class="hint-text">No mood selected yet</span>
            <button id="saveMoodBtn" type="button" class="btn-primary">Save Mood</button>
          </div>
        </div>

        <h3>Saved Mood History</h3>
        <div id="moodHistoryList" class="entries-grid"></div>
      </section>

      <!-- DIARY -->
      <section id="tab-diary" class="tab-panel">
        <div class="panel-header">
          <div>
            <h2>📔 Diary &amp; Journal</h2>
            <p>Write about what is happening in your day. Everything saves with the exact date and time.</p>
          </div>
        </div>

        <form id="diaryForm" class="card stack-form">
          <div class="form-row">
            <input type="text" id="diaryTitle" placeholder="Diary title (e.g., Today's reflections)..." required />
            <select id="diaryTag">
              <option value="Daily Life">Daily Life</option>
              <option value="Reflection">Reflection</option>
              <option value="Gratitude">Gratitude</option>
              <option value="Goals">Goals</option>
              <option value="Milestone">Milestone</option>
            </select>
          </div>
          <textarea id="diaryContent" rows="5" placeholder="Dear Diary, today..." required></textarea>
          <div class="form-actions">
            <span class="hint-text">Automatically timestamped in real time</span>
            <button type="submit" class="btn-primary">Save Diary Entry</button>
          </div>
        </form>

        <div class="search-bar-wrap">
          <input type="search" id="diarySearch" placeholder="🔍 Search your diary entries..." />
        </div>

        <div id="diaryEntriesList" class="diary-list"></div>
      </section>

      <!-- ADVICE & QUOTES -->
      <section id="tab-advice" class="tab-panel">
        <div class="panel-header">
          <div>
            <h2>💡 Advice &amp; Motivation</h2>
            <p>Read inspiring quotes and upload your own motivational advice.</p>
          </div>
        </div>

        <div class="card spotlight-card">
          <span class="spotlight-label">✨ Quote Spotlight</span>
          <blockquote id="spotlightQuoteText">"Small daily improvements over time lead to stunning results."</blockquote>
          <div class="spotlight-footer">
            <cite id="spotlightQuoteAuthor">— Robin Sharma</cite>
            <button id="nextSpotlightBtn" type="button" class="btn-secondary small">Next Quote →</button>
          </div>
        </div>

        <form id="adviceForm" class="card stack-form">
          <h3>Upload Motivational Quote or Advice</h3>
          <textarea id="adviceText" rows="3" placeholder="Write a motivational quote or piece of advice..." required></textarea>
          <div class="form-row">
            <input type="text" id="adviceAuthor" placeholder="Author or your name (optional)" />
            <select id="adviceCategory">
              <option value="Discipline">Discipline</option>
              <option value="Mindset">Mindset</option>
              <option value="Positivity">Positivity</option>
              <option value="Growth">Growth</option>
              <option value="Health">Health</option>
            </select>
            <button type="submit" class="btn-primary">Upload Quote</button>
          </div>
        </form>

        <div class="filter-bar">
          <h3>Quotes &amp; Advice Feed</h3>
          <select id="adviceFilterCategory">
            <option value="All">All Categories</option>
            <option value="Discipline">Discipline</option>
            <option value="Mindset">Mindset</option>
            <option value="Positivity">Positivity</option>
            <option value="Growth">Growth</option>
            <option value="Health">Health</option>
          </select>
        </div>

        <div id="adviceList" class="advice-grid"></div>
      </section>

      <!-- ADMIN CONTROL CENTER -->
      <section id="tab-admin" class="tab-panel">
        <div class="panel-header">
          <div>
            <h2>🛡️ Admin Control Center</h2>
            <p>Owner dashboard to manage site announcements, featured quotes, storage diagnostics, and security.</p>
          </div>
          <button id="adminLockBtn" type="button" class="btn-danger-outline hidden">🔒 Lock Admin</button>
        </div>

        <!-- Admin PIN Lock Screen -->
        <div id="adminLockScreen" class="card admin-lock-box">
          <div style="font-size: 2.4rem; margin-bottom: 0.5rem;">🛡️</div>
          <h3>Owner Verification</h3>
          <p class="subtitle" style="margin-bottom: 1rem;">Enter your Admin PIN to unlock site controls. (Default PIN: <strong>1234</strong>)</p>
          <form id="adminUnlockForm" class="stack-form">
            <input type="password" id="adminPinInput" inputmode="numeric" maxlength="12" placeholder="Enter Admin PIN..." required style="text-align:center; font-size:1.1rem; letter-spacing:0.2em;" />
            <button type="submit" class="btn-primary full-width">Unlock Admin Dashboard</button>
          </form>
        </div>

        <!-- Unlocked Admin Dashboard -->
        <div id="adminDashboardContent" class="hidden">
          <div class="stats-grid">
            <div class="card stat-card">
              <span class="stat-label">Total Habits / Check-ins</span>
              <strong id="adminStatHabits" class="stat-value">0 / 0</strong>
            </div>
            <div class="card stat-card">
              <span class="stat-label">Moods / Diary / Academic</span>
              <strong id="adminStatEntries" class="stat-value">0 / 0 / 0</strong>
            </div>
            <div class="card stat-card">
              <span class="stat-label">Quotes Published</span>
              <strong id="adminStatQuotes" class="stat-value">0</strong>
            </div>
            <div class="card stat-card">
              <span class="stat-label">Local Storage Used</span>
              <strong id="adminStatStorage" class="stat-value">0 KB</strong>
            </div>
          </div>

          <div class="charts-grid">
            <div class="card stack-form">
              <h3>📢 Top Site Announcement Banner</h3>
              <p class="hint-text" style="margin-top:0;">Publish a live motivation banner or reminder across the top of the app.</p>
              <input type="text" id="adminBannerInput" placeholder="e.g., Focus Week: Complete 100% of habits 5 days in a row!" />
              <div class="form-row">
                <button type="button" id="saveAdminBannerBtn" class="btn-primary">Publish Banner</button>
                <button type="button" id="clearAdminBannerBtn" class="btn-secondary">Clear Banner</button>
              </div>
            </div>

            <div class="card stack-form">
              <h3>🔐 Security &amp; Quick Tools</h3>
              <div class="form-row">
                <input type="password" id="newAdminPinInput" inputmode="numeric" maxlength="12" placeholder="New Admin PIN (min 4 digits)" />
                <button type="button" id="changeAdminPinBtn" class="btn-secondary">Update PIN</button>
              </div>
              <div class="form-row">
                <button type="button" id="adminLoadDemoBtn" class="btn-secondary small">✨ Load Sample Data</button>
                <button type="button" id="adminExportAllBtn" class="btn-secondary small">⬇ Export Full JSON</button>
                <button type="button" id="adminResetDataBtn" class="btn-danger-outline">⚠️ Reset All App Data</button>
              </div>
            </div>
          </div>

          <div class="card">
            <h3>💡 Manage &amp; Pin Quotes</h3>
            <p class="hint-text" style="margin-top:0; margin-bottom:0.8rem;">Pin any quote straight to the Spotlight banner or remove unwanted quotes.</p>
            <div id="adminQuotesList"></div>
          </div>
        </div>
      </section>
    </main>
  </div>

  <!-- UNIVERSAL EDIT ENTRY MODAL -->
  <div id="editEntryModal" class="modal-backdrop hidden">
    <div class="modal-card">
      <div class="modal-head">
        <h3 id="editModalTitle">Edit Entry</h3>
        <button id="closeEditModal" type="button" class="icon-btn">✕</button>
      </div>
      <form id="editEntryForm" class="stack-form">
        <div id="editModalFields" class="stack-form"></div>
        <div class="modal-actions">
          <button type="button" id="cancelEditModalBtn" class="btn-secondary">Cancel</button>
          <button type="submit" class="btn-primary">Save Changes</button>
        </div>
      </form>
    </div>
  </div>

  <!-- PROFILE, BACKUP & CLOUD SETTINGS MODAL -->
  <div id="profileModal" class="modal-backdrop hidden">
    <div class="modal-card">
      <div class="modal-head">
        <h3>Profile &amp; Sync Settings</h3>
        <button id="closeProfileModal" type="button" class="icon-btn">✕</button>
      </div>

      <form id="editProfileForm" class="stack-form">
        <div class="avatar-upload-box">
          <img id="editAvatarPreview" src="" alt="Profile Picture" />
          <label for="editAvatarInput" class="btn-secondary small">Change Picture</label>
          <input type="file" id="editAvatarInput" accept="image/*" hidden />
        </div>
        <div class="form-group">
          <label for="editName">Display Name</label>
          <input type="text" id="editName" required />
        </div>
        <div class="form-group">
          <label for="editEmail">Google Email</label>
          <input type="email" id="editEmail" required />
        </div>
        <div class="modal-actions">
          <button type="button" id="signOutBtn" class="btn-danger-outline">Sign Out / Switch</button>
          <button type="submit" class="btn-primary">Save Profile</button>
        </div>
      </form>

      <hr class="modal-divider" />

      <div class="stack-form">
        <h3>Backup Your Data</h3>
        <div class="form-row">
          <button type="button" id="exportBackupBtn" class="btn-secondary small">⬇ Download Backup (.json)</button>
          <label for="importBackupInput" class="btn-secondary small">⬆ Restore Backup</label>
          <input type="file" id="importBackupInput" accept="application/json" hidden />
        </div>
      </div>

      <hr class="modal-divider" />

      <form id="firebaseConfigForm" class="stack-form">
        <h3>Optional: Live Google Cloud Sync</h3>
        <p class="hint-text" style="margin-top:0;">Paste your Firebase <code>firebaseConfig</code> JSON here to enable Google Sign-In &amp; multi-device cloud sync.</p>
        <textarea id="firebaseConfigInput" rows="4" placeholder='{"apiKey":"...","authDomain":"...","projectId":"..."}'></textarea>
        <button type="submit" class="btn-secondary">Save &amp; Connect Firebase</button>
      </form>
    </div>
  </div>

  <div id="toast" class="toast hidden"></div>

  <script>
    const STORAGE_KEY = "lifepulse_app_data_v1";
    const FIREBASE_CFG_KEY = "lifepulse_firebase_cfg_v1";
    const THEME_KEY = "lifepulse_theme_v1";

    const SECTION_TITLES = {
      home: "🏠 Home Overview",
      habits: "✅ Habits",
      academics: "🎓 Academics",
      calendar: "📅 Calendar",
      analytics: "📊 Analytics",
      mood: "😊 Mood",
      diary: "📔 Diary",
      advice: "💡 Advice & Quotes",
      admin: "🛡️ Admin"
    };

    const MOOD_CHOICES = [
      { mood: "Happy", emoji: "😄" },
      { mood: "Calm", emoji: "😌" },
      { mood: "Motivated", emoji: "🔥" },
      { mood: "Grateful", emoji: "🙏" },
      { mood: "Tired", emoji: "😴" },
      { mood: "Stressed", emoji: "😰" },
      { mood: "Sad", emoji: "😢" },
      { mood: "Anxious", emoji: "😟" }
    ];

    const DEFAULT_QUOTES = [
      { id: "q1", text: "Small daily improvements over time lead to stunning results.", author: "Robin Sharma", category: "Discipline", likes: 12, liked: false, createdAt: new Date().toISOString() },
      { id: "q2", text: "You do not rise to the level of your goals. You fall to the level of your systems.", author: "James Clear", category: "Growth", likes: 19, liked: false, createdAt: new Date().toISOString() },
      { id: "q3", text: "Be gentle with yourself. You are doing the best you can with what you have today.", author: "Unknown", category: "Positivity", likes: 9, liked: false, createdAt: new Date().toISOString() },
      { id: "q4", text: "Discipline is choosing between what you want now and what you want most.", author: "Abraham Lincoln", category: "Mindset", likes: 15, liked: false, createdAt: new Date().toISOString() }
    ];

    let appData = {
      user: null,
      habits: [],
      academics: { units: [], assessments: [] },
      moods: [],
      diary: [],
      advice: [...DEFAULT_QUOTES],
      announcement: "",
      adminPin: "1234"
    };

    let auth = null;
    let db = null;
    let isFirebaseConfigured = false;
    let firestoreUnsubscribe = null;

    let currentView = "home";
    let selectedDateKey = formatLocalDateKey(new Date());
    let calendarViewDate = new Date();
    let calendarSelectedDateKey = formatLocalDateKey(new Date());
    let selectedMood = null;
    let spotlightIndex = 0;
    let pendingAuthAvatar = null;
    let pendingEditAvatar = null;
    let editingContext = null;
    let isAdminUnlocked = false;

    let weeklyChartInstance = null;
    let breakdownChartInstance = null;
    let moodChartInstance = null;
    let unitComparisonChartInstance = null;
    let academicTrendChartInstance = null;

    function initFirebaseIfAvailable() {
      const savedCfgRaw = localStorage.getItem(FIREBASE_CFG_KEY);
      if (!savedCfgRaw || typeof firebase === "undefined") return;
      try {
        const cfg = JSON.parse(savedCfgRaw);
        if (cfg && cfg.apiKey && cfg.projectId) {
          if (!firebase.apps.length) firebase.initializeApp(cfg);
          auth = firebase.auth();
          db = firebase.firestore();
          isFirebaseConfigured = true;
        }
      } catch (e) {
        console.warn("Invalid saved Firebase config:", e);
      }
    }

    function formatLocalDateKey(date) {
      const y = date.getFullYear();
      const m = String(date.getMonth() + 1).padStart(2, "0");
      const d = String(date.getDate()).padStart(2, "0");
      return `${y}-${m}-${d}`;
    }

    function formatReadableDateTime(isoString) {
      return new Date(isoString).toLocaleString(undefined, {
        weekday: "short", month: "short", day: "numeric", year: "numeric",
        hour: "2-digit", minute: "2-digit"
      });
    }

    function escapeHtml(str) {
      const div = document.createElement("div");
      div.textContent = str ?? "";
      return div.innerHTML;
    }

    function generateId(prefix = "id") {
      return `${prefix}_${Date.now()}_${Math.random().toString(36).slice(2, 7)}`;
    }

    function showToast(message) {
      const toast = document.getElementById("toast");
      if (!toast) return;
      toast.textContent = message;
      toast.classList.remove("hidden");
      clearTimeout(showToast._timer);
      showToast._timer = setTimeout(() => toast.classList.add("hidden"), 2800);
    }

    /* THEME MANAGEMENT */
    function applyTheme(theme) {
      const isDark = theme === "dark";
      document.body.classList.toggle("dark-mode", isDark);
      const iconChar = isDark ? "☀️" : "🌙";
      const icon1 = document.getElementById("themeToggleIcon");
      const icon2 = document.getElementById("sectionThemeIcon");
      if (icon1) icon1.textContent = iconChar;
      if (icon2) icon2.textContent = iconChar;
      localStorage.setItem(THEME_KEY, isDark ? "dark" : "light");
      if (currentView === "analytics") renderAnalytics();
      if (currentView === "academics") renderAcademicCharts();
    }

    /* DRAWER & VIEW NAVIGATION */
    function openDrawer() {
      document.getElementById("navDrawer")?.classList.add("open");
      document.getElementById("drawerBackdrop")?.classList.remove("hidden");
    }

    function closeDrawer() {
      document.getElementById("navDrawer")?.classList.remove("open");
      document.getElementById("drawerBackdrop")?.classList.add("hidden");
    }

    function switchView(tab) {
      const target = SECTION_TITLES[tab] ? tab : "home";
      currentView = target;
      closeDrawer();

      // Highlight active item in drawer
      document.querySelectorAll(".drawer-item").forEach((btn) => {
        btn.classList.toggle("active", btn.dataset.navTarget === target);
      });

      // Toggle visible panel
      document.querySelectorAll(".tab-panel").forEach((panel) => {
        panel.classList.toggle("active", panel.id === `tab-${target}`);
      });

      // Toggle Home Header vs Section Back Topbar
      const homeHeader = document.getElementById("homeHeader");
      const sectionTopbar = document.getElementById("sectionTopbar");
      const sectionTitleEl = document.getElementById("sectionTopbarTitle");

      if (target === "home") {
        homeHeader?.classList.remove("hidden");
        sectionTopbar?.classList.add("hidden");
        renderHomeHubSummary();
      } else {
        homeHeader?.classList.add("hidden");
        sectionTopbar?.classList.remove("hidden");
        if (sectionTitleEl) sectionTitleEl.textContent = SECTION_TITLES[target];
      }

      window.scrollTo({ top: 0, behavior: "smooth" });

      if (target === "analytics") renderAnalytics();
      if (target === "calendar") renderCalendar();
      if (target === "academics") renderAcademics();
      if (target === "admin" && isAdminUnlocked) renderAdminDashboard();
    }

    window.switchTab = switchView;

    /* IMAGE COMPRESSION & STORAGE */
    function compressImageFile(file, callback) {
      const reader = new FileReader();
      reader.onload = (e) => {
        const img = new Image();
        img.onload = () => {
          const canvas = document.createElement("canvas");
          const maxSize = 220;
          let { width, height } = img;
          if (width > height && width > maxSize) {
            height = Math.round((height * maxSize) / width);
            width = maxSize;
          } else if (height > maxSize) {
            width = Math.round((width * maxSize) / height);
            height = maxSize;
          }
          canvas.width = width;
          canvas.height = height;
          canvas.getContext("2d").drawImage(img, 0, 0, width, height);
          callback(canvas.toDataURL("image/jpeg", 0.82));
        };
        img.src = e.target.result;
      };
      reader.readAsDataURL(file);
    }

    function loadLocalData() {
      const raw = localStorage.getItem(STORAGE_KEY);
      if (raw) {
        try {
          const parsed = JSON.parse(raw);
          appData = {
            ...appData,
            ...parsed,
            academics: {
              units: parsed.academics?.units || [],
              assessments: parsed.academics?.assessments || []
            },
            advice: parsed.advice && parsed.advice.length ? parsed.advice : [...DEFAULT_QUOTES],
            adminPin: parsed.adminPin || "1234"
          };
        } catch (err) {
          console.error(err);
        }
      }
    }

    async function saveData(reason = "Saved") {
      const syncBadge = document.getElementById("syncBadge");
      localStorage.setItem(STORAGE_KEY, JSON.stringify(appData));
      renderHomeHubSummary();

      if (db && appData.user && appData.user.mode === "google") {
        try {
          if (syncBadge) syncBadge.textContent = "● Syncing...";
          await db.collection("users").doc(appData.user.uid).set({
            user: appData.user,
            habits: appData.habits,
            academics: appData.academics,
            moods: appData.moods,
            diary: appData.diary,
            advice: appData.advice,
            announcement: appData.announcement || "",
            updatedAt: firebase.firestore.FieldValue.serverTimestamp()
          }, { merge: true });
          if (syncBadge) {
            syncBadge.textContent = `● Cloud Synced (${new Date().toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" })})`;
          }
        } catch (err) {
          if (syncBadge) syncBadge.textContent = "● Saved Locally";
        }
      } else if (syncBadge) {
        syncBadge.textContent = `● Saved (${new Date().toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" })})`;
      }
      if (reason) showToast(reason);
    }

    function startLiveClock() {
      const el = document.getElementById("liveClock");
      if (!el) return;
      const tick = () => {
        el.textContent = new Date().toLocaleString(undefined, {
          weekday: "short", month: "short", day: "numeric", year: "numeric",
          hour: "2-digit", minute: "2-digit", second: "2-digit"
        });
      };
      tick();
      setInterval(tick, 1000);
    }

    function initAuth() {
      const overlay = document.getElementById("authOverlay");
      const note = document.getElementById("firebaseStatusNote");

      if (isFirebaseConfigured && auth) {
        if (note) note.textContent = "Google Cloud Sync is connected.";
        auth.onAuthStateChanged((fbUser) => {
          if (fbUser) {
            appData.user = {
              uid: fbUser.uid,
              name: fbUser.displayName || "Google User",
              email: fbUser.email || "",
              photoURL: (appData.user?.uid === fbUser.uid && appData.user?.photoURL) || fbUser.photoURL || `https://ui-avatars.com/api/?name=${encodeURIComponent(fbUser.displayName || "U")}&background=4f46e5&color=fff`,
              mode: "google"
            };
            overlay?.classList.add("hidden");
            subscribeToCloud(fbUser.uid);
            renderHeaderProfile();
          } else if (!appData.user) {
            overlay?.classList.remove("hidden");
          }
        });
      } else {
        if (note) note.textContent = "Works immediately offline/locally. You can connect Google Cloud Sync anytime in Profile Settings.";
        if (!appData.user) overlay?.classList.remove("hidden");
        else {
          overlay?.classList.add("hidden");
          renderHeaderProfile();
        }
      }
    }

    function subscribeToCloud(uid) {
      if (!db) return;
      if (firestoreUnsubscribe) firestoreUnsubscribe();
      firestoreUnsubscribe = db.collection("users").doc(uid).onSnapshot((doc) => {
        if (doc.exists) {
          const d = doc.data();
          if (d.habits) appData.habits = d.habits;
          if (d.academics) appData.academics = d.academics;
          if (d.moods) appData.moods = d.moods;
          if (d.diary) appData.diary = d.diary;
          if (d.advice?.length) appData.advice = d.advice;
          if (typeof d.announcement === "string") appData.announcement = d.announcement;
          if (d.user?.photoURL) appData.user.photoURL = d.user.photoURL;
          localStorage.setItem(STORAGE_KEY, JSON.stringify(appData));
          renderAll();
        } else {
          saveData("Google Cloud Sync active!");
        }
      });
    }

    function renderAnnouncementBanner() {
      const bar = document.getElementById("adminAnnouncementBar");
      const textEl = document.getElementById("adminAnnouncementText");
      if (!bar || !textEl) return;
      if (appData.announcement && appData.announcement.trim()) {
        textEl.textContent = `📢 ${appData.announcement}`;
        bar.classList.remove("hidden");
      } else {
        bar.classList.add("hidden");
      }
    }

    function renderHeaderProfile() {
      const greetingNameEl = document.getElementById("greetingUserName");
      const drawerNameEl = document.getElementById("drawerUserName");
      const drawerAvatarEl = document.getElementById("drawerAvatar");
      const headerAvatar = document.getElementById("headerAvatar");
      const sectionAvatar = document.getElementById("sectionAvatar");
      const headerName = document.getElementById("headerName");
      const headerEmail = document.getElementById("headerEmail");

      if (!appData.user) {
        if (greetingNameEl) greetingNameEl.textContent = "Guest";
        if (drawerNameEl) drawerNameEl.textContent = "Guest User";
        return;
      }
      const displayName = appData.user.name || "Guest";
      if (greetingNameEl) greetingNameEl.textContent = displayName;
      if (drawerNameEl) drawerNameEl.textContent = displayName;
      if (drawerAvatarEl) drawerAvatarEl.src = appData.user.photoURL;
      if (headerAvatar) headerAvatar.src = appData.user.photoURL;
      if (sectionAvatar) sectionAvatar.src = appData.user.photoURL;
      if (headerName) headerName.textContent = displayName;
      if (headerEmail) headerEmail.textContent = appData.user.email;
    }

    function renderHomeHubSummary() {
      const todayKey = formatLocalDateKey(new Date());
      const doneToday = appData.habits.filter((h) => Boolean(h.completions[todayKey])).length;
      const hubHabit = document.getElementById("hubHabitSummary");
      if (hubHabit) hubHabit.textContent = `${doneToday} / ${appData.habits.length}`;

      const allPcts = appData.academics.assessments.map((a) => (a.score / a.total) * 100);
      const meanPct = allPcts.length ? Math.round(allPcts.reduce((s, v) => s + v, 0) / allPcts.length) : 0;
      const hubAcad = document.getElementById("hubAcademicSummary");
      if (hubAcad) hubAcad.textContent = allPcts.length ? `${meanPct}%` : "0%";

      const hubMood = document.getElementById("hubMoodSummary");
      if (hubMood) hubMood.textContent = appData.moods.length;

      const hubDiary = document.getElementById("hubDiarySummary");
      if (hubDiary) hubDiary.textContent = appData.diary.length;
    }

    /* HABITS */
    function calculateStreak(habit) {
      let streak = 0;
      const cursor = new Date();
      if (!habit.completions[formatLocalDateKey(cursor)]) cursor.setDate(cursor.getDate() - 1);
      while (habit.completions[formatLocalDateKey(cursor)]) {
        streak++;
        cursor.setDate(cursor.getDate() - 1);
      }
      return streak;
    }

    function renderHabits() {
      const listEl = document.getElementById("habitsList");
      const dateInput = document.getElementById("habitDatePicker");
      if (dateInput) dateInput.value = selectedDateKey;
      if (!listEl) return;

      const total = appData.habits.length;
      const completedCount = appData.habits.filter((h) => Boolean(h.completions[selectedDateKey])).length;
      const pct = total ? Math.round((completedCount / total) * 100) : 0;

      document.getElementById("todayProgressText").textContent = `${completedCount} of ${total} habits completed (${selectedDateKey})`;
      document.getElementById("todayProgressPct").textContent = `${pct}%`;
      document.getElementById("todayProgressBar").style.width = `${pct}%`;

      if (!total) {
        listEl.innerHTML = `<div class="empty-state">No habits yet. Write your first habit above to start tracking!</div>`;
        return;
      }

      listEl.innerHTML = appData.habits.map((h) => {
        const doneAt = h.completions[selectedDateKey];
        const streak = calculateStreak(h);
        return `
          <div class="habit-item ${doneAt ? "completed" : ""}">
            <div class="habit-left">
              <input type="checkbox" class="habit-checkbox" ${doneAt ? "checked" : ""} onchange="toggleHabit('${h.id}')" />
              <div class="habit-info">
                <h4>${escapeHtml(h.title)}</h4>
                <div class="habit-meta">
                  <span class="badge">${escapeHtml(h.category)}</span>
                  <span class="badge streak">🔥 ${streak} day streak</span>
                  ${doneAt ? `<span>✓ Ticked at ${new Date(doneAt).toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" })}</span>` : `<span>Not ticked yet</span>`}
                </div>
              </div>
            </div>
            <div class="action-group">
              <button class="icon-btn" type="button" title="Edit Habit" onclick="openEditModal('habit', '${h.id}')"><span class="anim-icon">✏️</span></button>
              <button class="icon-btn danger" type="button" title="Delete Habit" onclick="deleteHabit('${h.id}')"><span class="anim-icon">🗑</span></button>
            </div>
          </div>
        `;
      }).join("");
    }

    window.toggleHabit = (id) => {
      const h = appData.habits.find((x) => x.id === id);
      if (!h) return;
      if (h.completions[selectedDateKey]) {
        delete h.completions[selectedDateKey];
        saveData(`Unchecked "${h.title}"`);
      } else {
        h.completions[selectedDateKey] = new Date().toISOString();
        saveData(`Completed "${h.title}"!`);
      }
      renderHabits();
      renderCalendar();
      renderAnalytics();
    };

    window.deleteHabit = (id) => {
      appData.habits = appData.habits.filter((h) => h.id !== id);
      saveData("Habit deleted");
      renderHabits();
      renderCalendar();
      renderAnalytics();
    };

    /* ACADEMICS & TOPICAL REVISION ENGINE */
    function getGradeLetter(pct) {
      if (pct >= 70) return "A";
      if (pct >= 60) return "B";
      if (pct >= 50) return "C";
      if (pct >= 40) return "D";
      return "E";
    }

    function computeUnitStats(unit) {
      const items = appData.academics.assessments.filter((a) => a.unitId === unit.id);
      if (!items.length) {
        return { count: 0, overallAvg: null, caAvg: null, revisionAvg: null };
      }
      const pcts = items.map((a) => (a.score / a.total) * 100);
      const overallAvg = Math.round(pcts.reduce((s, v) => s + v, 0) / pcts.length);

      const caItems = items.filter((a) => ["CAT 1", "CAT 2", "Assignment", "Lab / Practical"].includes(a.type));
      const caAvg = caItems.length
        ? Math.round(caItems.reduce((s, a) => s + (a.score / a.total) * 100, 0) / caItems.length)
        : null;

      const revItems = items.filter((a) => a.type === "Topical Revision");
      const revisionAvg = revItems.length
        ? Math.round(revItems.reduce((s, a) => s + (a.score / a.total) * 100, 0) / revItems.length)
        : null;

      return { count: items.length, overallAvg, caAvg, revisionAvg };
    }

    function syncAcademicDropdowns() {
      const unitSelect = document.getElementById("assessUnitSelect");
      const filterSelect = document.getElementById("acadFilterUnit");
      if (!unitSelect || !filterSelect) return;

      const prevUnitVal = unitSelect.value;
      const prevFilterVal = filterSelect.value;

      unitSelect.innerHTML = `<option value="">-- Select Course Unit --</option>` +
        appData.academics.units.map((u) => `<option value="${u.id}">${escapeHtml(u.code)} — ${escapeHtml(u.name)}</option>`).join("");
      if (appData.academics.units.some((u) => u.id === prevUnitVal)) unitSelect.value = prevUnitVal;

      filterSelect.innerHTML = `<option value="All">All Units</option>` +
        appData.academics.units.map((u) => `<option value="${u.id}">${escapeHtml(u.code)} (${escapeHtml(u.name)})</option>`).join("");
      if (prevFilterVal === "All" || appData.academics.units.some((u) => u.id === prevFilterVal)) {
        filterSelect.value = prevFilterVal;
      }
    }

    function renderAcademics() {
      syncAcademicDropdowns();

      const units = appData.academics.units;
      const assessments = appData.academics.assessments;
      const query = (document.getElementById("acadSearchInput")?.value || "").trim().toLowerCase();
      const unitFilter = document.getElementById("acadFilterUnit")?.value || "All";
      const typeFilter = document.getElementById("acadFilterType")?.value || "All";

      const allPcts = assessments.map((a) => (a.score / a.total) * 100);
      const meanPct = allPcts.length ? Math.round(allPcts.reduce((s, v) => s + v, 0) / allPcts.length) : 0;
      document.getElementById("acadStatMean").textContent = allPcts.length ? `${meanPct}% (${getGradeLetter(meanPct)})` : "0%";
      document.getElementById("acadStatTotalCount").textContent = assessments.length;

      let bestUnitLabel = "—";
      let bestUnitAvg = -1;
      let unitsOnTarget = 0;

      units.forEach((u) => {
        const st = computeUnitStats(u);
        if (st.overallAvg !== null) {
          if (st.overallAvg > bestUnitAvg) {
            bestUnitAvg = st.overallAvg;
            bestUnitLabel = `${u.code} (${st.overallAvg}%)`;
          }
          if (st.overallAvg >= (Number(u.target) || 70)) unitsOnTarget++;
        }
      });

      document.getElementById("acadStatBestUnit").textContent = bestUnitLabel;
      document.getElementById("acadStatOnTarget").textContent = `${unitsOnTarget} / ${units.length}`;

      const filteredUnits = units.filter((u) => {
        if (unitFilter !== "All" && u.id !== unitFilter) return false;
        if (!query) return true;
        const unitMatch = u.code.toLowerCase().includes(query) ||
          u.name.toLowerCase().includes(query) ||
          (u.semester || "").toLowerCase().includes(query) ||
          (u.notes || "").toLowerCase().includes(query);
        const childMatch = assessments.some((a) => a.unitId === u.id && (
          a.topic.toLowerCase().includes(query) ||
          a.type.toLowerCase().includes(query) ||
          (a.note || "").toLowerCase().includes(query)
        ));
        return unitMatch || childMatch;
      });

      const unitsGrid = document.getElementById("unitsGrid");
      if (!filteredUnits.length) {
        unitsGrid.innerHTML = `<div class="empty-state" style="grid-column:1/-1;">No matching course units found. Add your semester units above!</div>`;
      } else {
        unitsGrid.innerHTML = filteredUnits.map((u) => {
          const st = computeUnitStats(u);
          const target = Number(u.target) || 70;
          const hasScores = st.overallAvg !== null;
          const isOnTrack = hasScores && st.overallAvg >= target;
          const cardStateClass = !hasScores ? "" : (isOnTrack ? "on-track" : "needs-focus");
          const statusBadge = !hasScores
            ? `<span class="badge">No scores yet</span>`
            : (isOnTrack
              ? `<span class="badge success">✓ On Track (Grade ${getGradeLetter(st.overallAvg)})</span>`
              : `<span class="badge warn">⚡ Needs Focus (Grade ${getGradeLetter(st.overallAvg)})</span>`);
          const barPct = hasScores ? Math.min(100, st.overallAvg) : 0;

          return `
            <div class="unit-card ${cardStateClass}">
              <div class="unit-card-top">
                <div>
                  <div class="habit-meta" style="margin-bottom:0.25rem;">
                    <span class="badge">${escapeHtml(u.code)}</span>
                    <span class="badge">${escapeHtml(u.semester || "Sem 1")}</span>
                    ${statusBadge}
                  </div>
                  <h4>${escapeHtml(u.name)}</h4>
                </div>
                <div class="action-group">
                  <button class="icon-btn" type="button" title="Edit Unit" onclick="openEditModal('unit', '${u.id}')"><span class="anim-icon">✏️</span></button>
                  <button class="icon-btn danger" type="button" title="Delete Unit" onclick="deleteAcademicUnit('${u.id}')"><span class="anim-icon">🗑</span></button>
                </div>
              </div>

              <div class="unit-metrics-row">
                <div class="unit-metric">
                  <span>Overall Avg</span>
                  <strong>${hasScores ? st.overallAvg + "%" : "—"}</strong>
                </div>
                <div class="unit-metric">
                  <span>CATs &amp; Assign</span>
                  <strong>${st.caAvg !== null ? st.caAvg + "%" : "—"}</strong>
                </div>
                <div class="unit-metric">
                  <span>Topical Rev</span>
                  <strong>${st.revisionAvg !== null ? st.revisionAvg + "%" : "—"}</strong>
                </div>
              </div>

              <div>
                <div class="progress-top" style="font-size:0.78rem; margin-bottom:0.3rem;">
                  <span>Progress (${st.count} logged)</span>
                  <span>Target: ${target}%</span>
                </div>
                <div class="progress-track" style="height:8px;">
                  <div class="progress-fill" style="width:${barPct}%;"></div>
                </div>
              </div>
              ${u.notes ? `<p class="hint-text" style="margin-top:0;">📌 ${escapeHtml(u.notes)}</p>` : ""}
            </div>
          `;
        }).join("");
      }

      const filteredAssessments = [...assessments]
        .sort((a, b) => new Date(b.dateKey || b.createdAt) - new Date(a.dateKey || a.createdAt))
        .filter((a) => {
          if (unitFilter !== "All" && a.unitId !== unitFilter) return false;
          if (typeFilter !== "All") {
            if (typeFilter === "CAT" && !a.type.startsWith("CAT")) return false;
            if (typeFilter !== "CAT" && a.type !== typeFilter) return false;
          }
          if (!query) return true;
          const unit = units.find((u) => u.id === a.unitId);
          const unitText = unit ? `${unit.code} ${unit.name}`.toLowerCase() : "";
          return a.topic.toLowerCase().includes(query) ||
            a.type.toLowerCase().includes(query) ||
            (a.note || "").toLowerCase().includes(query) ||
            unitText.includes(query);
        });

      const histEl = document.getElementById("assessmentsHistoryList");
      if (!filteredAssessments.length) {
        histEl.innerHTML = `<div class="empty-state">No CATs, assignments, or topical revision results match your filter yet.</div>`;
      } else {
        histEl.innerHTML = filteredAssessments.map((a) => {
          const unit = units.find((u) => u.id === a.unitId);
          const pct = Math.round((a.score / a.total) * 100);
          const grade = getGradeLetter(pct);
          const pillClass = pct >= 70 ? "high" : (pct >= 50 ? "mid" : "low");
          return `
            <div class="assessment-item">
              <div style="flex:1; min-width:210px;">
                <div class="habit-meta" style="margin-bottom:0.25rem;">
                  <span class="badge">${unit ? escapeHtml(unit.code) : "Unit"}</span>
                  <span class="badge">${escapeHtml(a.type)}</span>
                  <span>📅 ${escapeHtml(a.dateKey || formatLocalDateKey(new Date(a.createdAt)))}</span>
                </div>
                <h4 style="font-size:1rem;">${escapeHtml(a.topic)}</h4>
                ${a.note ? `<p class="hint-text" style="margin-top:0.2rem;">💡 ${escapeHtml(a.note)}</p>` : ""}
              </div>
              <div style="display:flex; align-items:center; gap:0.75rem;">
                <span class="score-pill ${pillClass}">${a.score}/${a.total} (${pct}% · ${grade})</span>
                <div class="action-group">
                  <button class="icon-btn" type="button" title="Edit Result" onclick="openEditModal('assessment', '${a.id}')"><span class="anim-icon">✏️</span></button>
                  <button class="icon-btn danger" type="button" title="Delete Result" onclick="deleteAssessment('${a.id}')"><span class="anim-icon">🗑</span></button>
                </div>
              </div>
            </div>
          `;
        }).join("");
      }

      const weakEl = document.getElementById("academicWeaknessList");
      const weakTopics = assessments
        .map((a) => {
          const unit = units.find((u) => u.id === a.unitId);
          const target = unit ? (Number(unit.target) || 70) : 70;
          const pct = Math.round((a.score / a.total) * 100);
          return { ...a, unit, target, pct };
        })
        .filter((item) => item.pct < 60 || item.pct < item.target)
        .sort((a, b) => a.pct - b.pct);

      if (!assessments.length) {
        weakEl.innerHTML = `<p class="hint-text">Log a few CATs or topical revision questions to unlock automatic topic weakness analysis.</p>`;
      } else if (!weakTopics.length) {
        weakEl.innerHTML = `<div class="inspector-item" style="border-left:4px solid var(--success);">🌟 <strong>Awesome work!</strong> Every recorded CAT, assignment, and topical revision is currently meeting or beating your target!</div>`;
      } else {
        weakEl.innerHTML = weakTopics.slice(0, 5).map((w) => `
          <div class="weakness-item">
            <div>
              <strong>${w.unit ? escapeHtml(w.unit.code) : "Unit"} — ${escapeHtml(w.topic)}</strong>
              <span class="badge">${escapeHtml(w.type)}</span>
              ${w.note ? `<div class="hint-text" style="margin-top:0.15rem;">Note: ${escapeHtml(w.note)}</div>` : ""}
            </div>
            <span class="badge danger">Scored ${w.pct}% (Target ${w.target}%) · Revise Topic</span>
          </div>
        `).join("");
      }

      renderAcademicCharts();
    }

    function renderAcademicCharts() {
      if (typeof Chart === "undefined") return;
      const isDark = document.body.classList.contains("dark-mode");
      const textColor = isDark ? "#cbd5e1" : "#475569";
      const gridColor = isDark ? "rgba(148, 163, 184, 0.15)" : "rgba(15, 23, 42, 0.08)";

      const units = appData.academics.units;
      const unitLabels = units.length ? units.map((u) => u.code) : ["No Units"];
      const unitAvgs = units.length ? units.map((u) => computeUnitStats(u).overallAvg ?? 0) : [0];
      const unitTargets = units.length ? units.map((u) => Number(u.target) || 70) : [70];

      if (unitComparisonChartInstance) unitComparisonChartInstance.destroy();
      const compCanvas = document.getElementById("unitComparisonChart");
      if (compCanvas) {
        unitComparisonChartInstance = new Chart(compCanvas.getContext("2d"), {
          type: "bar",
          data: {
            labels: unitLabels,
            datasets: [
              { label: "Current Avg (%)", data: unitAvgs, backgroundColor: "#4f46e5", borderRadius: 6 },
              { label: "Target (%)", data: unitTargets, backgroundColor: "rgba(16, 185, 129, 0.45)", borderRadius: 6 }
            ]
          },
          options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: { labels: { color: textColor } } },
            scales: {
              x: { ticks: { color: textColor }, grid: { color: gridColor } },
              y: { beginAtZero: true, max: 100, ticks: { color: textColor }, grid: { color: gridColor } }
            }
          }
        });
      }

      const sortedChrono = [...appData.academics.assessments].sort(
        (a, b) => new Date(a.dateKey || a.createdAt) - new Date(b.dateKey || b.createdAt)
      );
      const trendLabels = sortedChrono.length
        ? sortedChrono.slice(-12).map((a) => `${a.topic.slice(0, 14)} (${a.type})`)
        : ["No Scores Yet"];
      const trendData = sortedChrono.length
        ? sortedChrono.slice(-12).map((a) => Math.round((a.score / a.total) * 100))
        : [0];

      if (academicTrendChartInstance) academicTrendChartInstance.destroy();
      const trendCanvas = document.getElementById("academicTrendChart");
      if (trendCanvas) {
        academicTrendChartInstance = new Chart(trendCanvas.getContext("2d"), {
          type: "line",
          data: {
            labels: trendLabels,
            datasets: [{
              label: "Score (%)",
              data: trendData,
              borderColor: "#10b981",
              backgroundColor: "rgba(16, 185, 129, 0.16)",
              fill: true,
              tension: 0.32,
              pointRadius: 4
            }]
          },
          options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: { labels: { color: textColor } } },
            scales: {
              x: { ticks: { color: textColor }, grid: { color: gridColor } },
              y: { beginAtZero: true, max: 100, ticks: { color: textColor }, grid: { color: gridColor } }
            }
          }
        });
      }
    }

    window.deleteAcademicUnit = (id) => {
      appData.academics.units = appData.academics.units.filter((u) => u.id !== id);
      appData.academics.assessments = appData.academics.assessments.filter((a) => a.unitId !== id);
      saveData("Course unit and its results deleted");
      renderAcademics();
      renderCalendar();
    };

    window.deleteAssessment = (id) => {
      appData.academics.assessments = appData.academics.assessments.filter((a) => a.id !== id);
      saveData("Assessment result deleted");
      renderAcademics();
      renderCalendar();
    };

    /* INTERACTIVE CALENDAR & DAY INSPECTOR */
    function renderCalendar() {
      const grid = document.getElementById("calendarGrid");
      const monthLabel = document.getElementById("calMonthLabel");
      if (!grid || !monthLabel) return;

      const year = calendarViewDate.getFullYear();
      const month = calendarViewDate.getMonth();
      monthLabel.textContent = new Date(year, month, 1).toLocaleDateString(undefined, { month: "long", year: "numeric" });

      const firstDayIndex = new Date(year, month, 1).getDay();
      const daysInMonth = new Date(year, month + 1, 0).getDate();
      const todayKey = formatLocalDateKey(new Date());

      let cellsHtml = "";
      for (let i = 0; i < firstDayIndex; i++) {
        cellsHtml += `<div class="cal-day empty"></div>`;
      }

      for (let day = 1; day <= daysInMonth; day++) {
        const dObj = new Date(year, month, day);
        const key = formatLocalDateKey(dObj);
        const hasHabit = appData.habits.some((h) => Boolean(h.completions[key]));
        const hasAcademic = appData.academics.assessments.some((a) => (a.dateKey || formatLocalDateKey(new Date(a.createdAt))) === key);
        const hasMood = appData.moods.some((m) => (m.dateKey || formatLocalDateKey(new Date(m.createdAt))) === key);
        const hasDiary = appData.diary.some((d) => (d.dateKey || formatLocalDateKey(new Date(d.createdAt))) === key);

        const isToday = key === todayKey ? "today" : "";
        const isSelected = key === calendarSelectedDateKey ? "selected" : "";

        cellsHtml += `
          <button type="button" class="cal-day ${isToday} ${isSelected}" onclick="selectCalendarDate('${key}')">
            <span>${day}</span>
            <div class="cal-dots">
              ${hasHabit ? `<i class="cal-dot habit" title="Habit completed"></i>` : ""}
              ${hasAcademic ? `<i class="cal-dot academic" title="CAT / Assignment / Revision"></i>` : ""}
              ${hasMood ? `<i class="cal-dot mood" title="Mood logged"></i>` : ""}
              ${hasDiary ? `<i class="cal-dot diary" title="Diary written"></i>` : ""}
            </div>
          </button>
        `;
      }

      grid.innerHTML = cellsHtml;
      renderDayInspector();
    }

    window.selectCalendarDate = (dateKey) => {
      calendarSelectedDateKey = dateKey;
      renderCalendar();
    };

    function renderDayInspector() {
      const titleEl = document.getElementById("inspectorDateTitle");
      const contentEl = document.getElementById("dayInspectorContent");
      if (!titleEl || !contentEl) return;

      const [y, m, d] = calendarSelectedDateKey.split("-").map(Number);
      const readableDate = new Date(y, m - 1, d).toLocaleDateString(undefined, {
        weekday: "long", month: "long", day: "numeric", year: "numeric"
      });
      titleEl.textContent = readableDate;

      const completedHabits = appData.habits.filter((h) => Boolean(h.completions[calendarSelectedDateKey]));
      const dayAssessments = appData.academics.assessments.filter((a) => (a.dateKey || formatLocalDateKey(new Date(a.createdAt))) === calendarSelectedDateKey);
      const dayMoods = appData.moods.filter((item) => (item.dateKey || formatLocalDateKey(new Date(item.createdAt))) === calendarSelectedDateKey);
      const dayDiary = appData.diary.filter((item) => (item.dateKey || formatLocalDateKey(new Date(item.createdAt))) === calendarSelectedDateKey);

      const habitsHtml = completedHabits.length
        ? completedHabits.map((h) => `
            <div class="inspector-item">
              ✅ <strong>${escapeHtml(h.title)}</strong> <span class="badge">${escapeHtml(h.category)}</span>
              <div class="entry-time">Completed at ${new Date(h.completions[calendarSelectedDateKey]).toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" })}</div>
            </div>
          `).join("")
        : `<p class="hint-text">No habits ticked on this date.</p>`;

      const academicsHtml = dayAssessments.length
        ? dayAssessments.map((a) => {
            const unit = appData.academics.units.find((u) => u.id === a.unitId);
            const pct = Math.round((a.score / a.total) * 100);
            return `
              <div class="inspector-item">
                🎓 <strong>${unit ? escapeHtml(unit.code) : "Unit"}: ${escapeHtml(a.topic)}</strong>
                <span class="badge">${escapeHtml(a.type)} · ${a.score}/${a.total} (${pct}%)</span>
                ${a.note ? `<p style="margin-top:0.2rem;">${escapeHtml(a.note)}</p>` : ""}
              </div>
            `;
          }).join("")
        : `<p class="hint-text">No CATs, assignments, or topical revisions logged on this date.</p>`;

      const moodsHtml = dayMoods.length
        ? dayMoods.map((item) => `
            <div class="inspector-item">
              <strong>${escapeHtml(item.emoji)} ${escapeHtml(item.mood)}</strong>
              <p style="margin-top:0.2rem;">${escapeHtml(item.note || "No note written.")}</p>
              <div class="entry-time">${new Date(item.createdAt).toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" })}</div>
            </div>
          `).join("")
        : `<p class="hint-text">No mood logged on this date.</p>`;

      const diaryHtml = dayDiary.length
        ? dayDiary.map((item) => `
            <div class="inspector-item">
              <strong>📔 ${escapeHtml(item.title)}</strong> <span class="badge">${escapeHtml(item.tag)}</span>
              <p style="margin-top:0.3rem; white-space:pre-wrap;">${escapeHtml(item.content)}</p>
            </div>
          `).join("")
        : `<p class="hint-text">No diary entry written on this date.</p>`;

      contentEl.innerHTML = `
        <div class="inspector-section">
          <h4>✅ Completed Habits (${completedHabits.length})</h4>
          ${habitsHtml}
        </div>
        <div class="inspector-section">
          <h4>🎓 Academic Assessments &amp; Revision (${dayAssessments.length})</h4>
          ${academicsHtml}
        </div>
        <div class="inspector-section">
          <h4>😊 Moods Logged (${dayMoods.length})</h4>
          ${moodsHtml}
        </div>
        <div class="inspector-section">
          <h4>📔 Diary Entries (${dayDiary.length})</h4>
          ${diaryHtml}
        </div>
      `;
    }

    /* ANALYTICS & MOOD CHARTS */
    function renderAnalytics() {
      const totalHabits = appData.habits.length;
      const todayKey = formatLocalDateKey(new Date());
      const todayDone = appData.habits.filter((h) => Boolean(h.completions[todayKey])).length;
      const todayRate = totalHabits ? Math.round((todayDone / totalHabits) * 100) : 0;

      let bestStreak = 0, totalCheckins = 0;
      const logs = [];
      appData.habits.forEach((h) => {
        const s = calculateStreak(h);
        if (s > bestStreak) bestStreak = s;
        Object.entries(h.completions).forEach(([dateKey, timestamp]) => {
          totalCheckins++;
          logs.push({ title: h.title, category: h.category, timestamp });
        });
      });

      const elTodayRate = document.getElementById("statTodayRate");
      const elStreak = document.getElementById("statStreak");
      const elCheckins = document.getElementById("statTotalCheckins");
      const elMoods = document.getElementById("statTotalMoods");
      if (elTodayRate) elTodayRate.textContent = `${todayRate}%`;
      if (elStreak) elStreak.textContent = `${bestStreak} days`;
      if (elCheckins) elCheckins.textContent = totalCheckins;
      if (elMoods) elMoods.textContent = appData.moods.length;

      const labels = [], rates = [];
      for (let i = 6; i >= 0; i--) {
        const d = new Date();
        d.setDate(d.getDate() - i);
        const key = formatLocalDateKey(d);
        labels.push(d.toLocaleDateString(undefined, { weekday: "short", month: "short", day: "numeric" }));
        rates.push(totalHabits ? Math.round((appData.habits.filter((h) => h.completions[key]).length / totalHabits) * 100) : 0);
      }

      const isDark = document.body.classList.contains("dark-mode");
      const textColor = isDark ? "#cbd5e1" : "#475569";
      const gridColor = isDark ? "rgba(148, 163, 184, 0.15)" : "rgba(15, 23, 42, 0.08)";

      try {
        if (typeof Chart !== "undefined") {
          const weeklyCanvas = document.getElementById("weeklyChart");
          if (weeklyCanvas) {
            if (weeklyChartInstance) weeklyChartInstance.destroy();
            weeklyChartInstance = new Chart(weeklyCanvas.getContext("2d"), {
              type: "line",
              data: {
                labels,
                datasets: [{ label: "Completion %", data: rates, borderColor: "#6366f1", backgroundColor: "rgba(99,102,241,0.18)", fill: true, tension: 0.35, pointRadius: 4 }]
              },
              options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: { legend: { labels: { color: textColor } } },
                scales: {
                  x: { ticks: { color: textColor }, grid: { color: gridColor } },
                  y: { beginAtZero: true, max: 100, ticks: { color: textColor }, grid: { color: gridColor } }
                }
              }
            });
          }

          const breakdownCanvas = document.getElementById("habitBreakdownChart");
          if (breakdownCanvas) {
            if (breakdownChartInstance) breakdownChartInstance.destroy();
            breakdownChartInstance = new Chart(breakdownCanvas.getContext("2d"), {
              type: "bar",
              data: {
                labels: appData.habits.map((h) => h.title.slice(0, 18)),
                datasets: [{ label: "Total Days Completed", data: appData.habits.map((h) => Object.keys(h.completions).length), backgroundColor: "#10b981", borderRadius: 6 }]
              },
              options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: { legend: { labels: { color: textColor } } },
                scales: {
                  x: { ticks: { color: textColor }, grid: { color: gridColor } },
                  y: { beginAtZero: true, ticks: { stepSize: 1, color: textColor }, grid: { color: gridColor } }
                }
              }
            });
          }

          const moodCanvas = document.getElementById("moodTrendChart");
          if (moodCanvas) {
            const moodCounts = {};
            appData.moods.forEach((m) => {
              const key = `${m.emoji} ${m.mood}`;
              moodCounts[key] = (moodCounts[key] || 0) + 1;
            });
            const moodLabels = Object.keys(moodCounts).length ? Object.keys(moodCounts) : ["No moods yet"];
            const moodValues = Object.keys(moodCounts).length ? Object.values(moodCounts) : [1];
            const moodColors = Object.keys(moodCounts).length
              ? ["#6366f1", "#10b981", "#f59e0b", "#0ea5e9", "#ec4899", "#8b5cf6", "#ef4444", "#64748b"]
              : [gridColor];

            if (moodChartInstance) moodChartInstance.destroy();
            moodChartInstance = new Chart(moodCanvas.getContext("2d"), {
              type: "doughnut",
              data: {
                labels: moodLabels,
                datasets: [{ data: moodValues, backgroundColor: moodColors, borderWidth: 0 }]
              },
              options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                  legend: { position: "bottom", labels: { color: textColor, boxWidth: 12 } }
                }
              }
            });
          }
        }
      } catch (err) {
        console.warn("Analytics chart render warning:", err);
      }

      const logEl = document.getElementById("recentActivityList");
      if (logEl) {
        logs.sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp));
        logEl.innerHTML = logs.length
          ? logs.slice(0, 8).map((l) => `<li><span>✅ <strong>${escapeHtml(l.title)}</strong> <small>(${escapeHtml(l.category)})</small></span><span class="entry-time">${formatReadableDateTime(l.timestamp)}</span></li>`).join("")
          : `<li>No habit check-ins recorded yet.</li>`;
      }
    }

    /* MOODS */
    function renderMoods() {
      const el = document.getElementById("moodHistoryList");
      if (!el) return;
      if (!appData.moods.length) {
        el.innerHTML = `<div class="empty-state">No moods logged yet. Pick your mood above and save how you feel!</div>`;
        return;
      }
      el.innerHTML = [...appData.moods].sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt)).map((m) => `
        <div class="entry-card">
          <div class="entry-top">
            <span class="entry-mood-title">${escapeHtml(m.emoji)} ${escapeHtml(m.mood)}</span>
            <div class="action-group">
              <button class="icon-btn" type="button" title="Edit Mood" onclick="openEditModal('mood', '${m.id}')"><span class="anim-icon">✏️</span></button>
              <button class="icon-btn danger" type="button" title="Delete Mood" onclick="deleteMood('${m.id}')"><span class="anim-icon">🗑</span></button>
            </div>
          </div>
          <p>${escapeHtml(m.note || "No additional note written.")}</p>
          <span class="entry-time">🕒 ${formatReadableDateTime(m.createdAt)}</span>
        </div>
      `).join("");
    }

    window.deleteMood = (id) => {
      appData.moods = appData.moods.filter((m) => m.id !== id);
      saveData("Mood entry removed");
      renderMoods();
      renderCalendar();
      renderAnalytics();
    };

    /* DIARY */
    function renderDiary() {
      const el = document.getElementById("diaryEntriesList");
      if (!el) return;
      const q = (document.getElementById("diarySearch")?.value || "").trim().toLowerCase();
      let list = [...appData.diary].sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
      if (q) list = list.filter((e) => e.title.toLowerCase().includes(q) || e.content.toLowerCase().includes(q) || e.tag.toLowerCase().includes(q));

      el.innerHTML = list.length ? list.map((e) => `
        <article class="diary-card">
          <div class="diary-card-head">
            <div>
              <h4>${escapeHtml(e.title)}</h4>
              <div class="habit-meta"><span class="badge">${escapeHtml(e.tag)}</span><span>📅 ${formatReadableDateTime(e.createdAt)}</span></div>
            </div>
            <div class="action-group">
              <button class="icon-btn" type="button" title="Edit Diary Entry" onclick="openEditModal('diary', '${e.id}')"><span class="anim-icon">✏️</span></button>
              <button class="icon-btn danger" type="button" title="Delete Diary Entry" onclick="deleteDiary('${e.id}')"><span class="anim-icon">🗑</span></button>
            </div>
          </div>
          <div class="diary-body">${escapeHtml(e.content)}</div>
        </article>
      `).join("") : `<div class="empty-state">No diary entries found. Start writing your journal above!</div>`;
    }

    window.deleteDiary = (id) => {
      appData.diary = appData.diary.filter((d) => d.id !== id);
      saveData("Diary entry deleted");
      renderDiary();
      renderCalendar();
    };

    /* UNIVERSAL EDIT ENTRY MODAL */
    window.openEditModal = (type, id) => {
      const modal = document.getElementById("editEntryModal");
      const titleEl = document.getElementById("editModalTitle");
      const fieldsEl = document.getElementById("editModalFields");
      editingContext = { type, id };

      if (type === "habit") {
        const h = appData.habits.find((x) => x.id === id);
        if (!h) return;
        titleEl.textContent = "Edit Habit";
        const cats = ["Health", "Productivity", "Mindfulness", "Learning", "Fitness"];
        fieldsEl.innerHTML = `
          <div class="form-group">
            <label for="editHabitTitle">Habit Name</label>
            <input type="text" id="editHabitTitle" value="${escapeHtml(h.title)}" required />
          </div>
          <div class="form-group">
            <label for="editHabitCat">Category</label>
            <select id="editHabitCat">
              ${cats.map((c) => `<option value="${c}" ${h.category === c ? "selected" : ""}>${c}</option>`).join("")}
            </select>
          </div>
        `;
      } else if (type === "mood") {
        const m = appData.moods.find((x) => x.id === id);
        if (!m) return;
        titleEl.textContent = "Edit Mood Entry";
        fieldsEl.innerHTML = `
          <div class="form-group">
            <label for="editMoodSelect">Mood</label>
            <select id="editMoodSelect">
              ${MOOD_CHOICES.map((mc) => `<option value="${mc.mood}|${mc.emoji}" ${m.mood === mc.mood ? "selected" : ""}>${mc.emoji} ${mc.mood}</option>`).join("")}
            </select>
          </div>
          <div class="form-group">
            <label for="editMoodNote">Note</label>
            <textarea id="editMoodNote" rows="3">${escapeHtml(m.note || "")}</textarea>
          </div>
        `;
      } else if (type === "diary") {
        const d = appData.diary.find((x) => x.id === id);
        if (!d) return;
        titleEl.textContent = "Edit Diary Entry";
        const tags = ["Daily Life", "Reflection", "Gratitude", "Goals", "Milestone"];
        fieldsEl.innerHTML = `
          <div class="form-group">
            <label for="editDiaryTitle">Title</label>
            <input type="text" id="editDiaryTitle" value="${escapeHtml(d.title)}" required />
          </div>
          <div class="form-group">
            <label for="editDiaryTag">Tag</label>
            <select id="editDiaryTag">
              ${tags.map((t) => `<option value="${t}" ${d.tag === t ? "selected" : ""}>${t}</option>`).join("")}
            </select>
          </div>
          <div class="form-group">
            <label for="editDiaryContent">Entry</label>
            <textarea id="editDiaryContent" rows="5" required>${escapeHtml(d.content)}</textarea>
          </div>
        `;
      } else if (type === "unit") {
        const u = appData.academics.units.find((x) => x.id === id);
        if (!u) return;
        titleEl.textContent = "Edit Course Unit";
        fieldsEl.innerHTML = `
          <div class="form-group">
            <label for="editUnitCode">Unit Code</label>
            <input type="text" id="editUnitCode" value="${escapeHtml(u.code)}" required />
          </div>
          <div class="form-group">
            <label for="editUnitName">Unit Name</label>
            <input type="text" id="editUnitName" value="${escapeHtml(u.name)}" required />
          </div>
          <div class="form-group">
            <label for="editUnitSemester">Semester</label>
            <input type="text" id="editUnitSemester" value="${escapeHtml(u.semester || "Sem 1")}" />
          </div>
          <div class="form-group">
            <label for="editUnitTarget">Target Percentage (%)</label>
            <input type="number" id="editUnitTarget" min="1" max="100" value="${Number(u.target) || 70}" required />
          </div>
          <div class="form-group">
            <label for="editUnitNotes">Notes</label>
            <input type="text" id="editUnitNotes" value="${escapeHtml(u.notes || "")}" />
          </div>
        `;
      } else if (type === "assessment") {
        const a = appData.academics.assessments.find((x) => x.id === id);
        if (!a) return;
        titleEl.textContent = "Edit CAT / Revision Result";
        const types = ["CAT 1", "CAT 2", "Assignment", "Topical Revision", "Lab / Practical"];
        fieldsEl.innerHTML = `
          <div class="form-group">
            <label for="editAssessType">Assessment Type</label>
            <select id="editAssessType">
              ${types.map((t) => `<option value="${t}" ${a.type === t ? "selected" : ""}>${t}</option>`).join("")}
            </select>
          </div>
          <div class="form-group">
            <label for="editAssessTopic">Topic / Title</label>
            <input type="text" id="editAssessTopic" value="${escapeHtml(a.topic)}" required />
          </div>
          <div class="form-row">
            <div class="form-group" style="flex:1;">
              <label for="editAssessScore">Marks Scored</label>
              <input type="number" step="0.1" min="0" id="editAssessScore" value="${a.score}" required />
            </div>
            <div class="form-group" style="flex:1;">
              <label for="editAssessTotal">Out of Total</label>
              <input type="number" step="0.1" min="1" id="editAssessTotal" value="${a.total}" required />
            </div>
          </div>
          <div class="form-group">
            <label for="editAssessDate">Date</label>
            <input type="date" id="editAssessDate" value="${escapeHtml(a.dateKey || formatLocalDateKey(new Date()))}" />
          </div>
          <div class="form-group">
            <label for="editAssessNote">Revision Notes</label>
            <textarea id="editAssessNote" rows="2">${escapeHtml(a.note || "")}</textarea>
          </div>
        `;
      }

      modal?.classList.remove("hidden");
    };

    /* ADVICE & QUOTES */
    function renderSpotlight() {
      if (!appData.advice.length) return;
      spotlightIndex %= appData.advice.length;
      const q = appData.advice[spotlightIndex];
      document.getElementById("spotlightQuoteText").textContent = `"${q.text}"`;
      document.getElementById("spotlightQuoteAuthor").textContent = `— ${q.author || "Anonymous"}`;
    }

    function renderAdvice() {
      renderSpotlight();
      const filter = document.getElementById("adviceFilterCategory")?.value || "All";
      const el = document.getElementById("adviceList");
      if (!el) return;
      let items = [...appData.advice].sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
      if (filter !== "All") items = items.filter((i) => i.category === filter);

      el.innerHTML = items.length ? items.map((i) => `
        <div class="advice-card">
          <div>
            <span class="badge">${escapeHtml(i.category)}</span>
            <p class="advice-quote" style="margin-top:0.6rem;">"${escapeHtml(i.text)}"</p>
          </div>
          <div class="advice-footer">
            <span>— ${escapeHtml(i.author || "Anonymous")}</span>
            <button class="like-btn ${i.liked ? "liked" : ""}" type="button" onclick="toggleLikeAdvice('${i.id}')">♥ ${i.likes || 0}</button>
          </div>
        </div>
      `).join("") : `<div class="empty-state">No quotes in this category yet. Upload one above!</div>`;
    }

    window.toggleLikeAdvice = (id) => {
      const item = appData.advice.find((q) => q.id === id);
      if (!item) return;
      item.liked = !item.liked;
      item.likes = (item.likes || 0) + (item.liked ? 1 : -1);
      saveData(item.liked ? "Liked quote!" : "Removed like");
      renderAdvice();
    };

    /* ADMIN CONTROL CENTER */
    function renderAdminDashboard() {
      if (!isAdminUnlocked) return;
      const totalCheckins = appData.habits.reduce((acc, h) => acc + Object.keys(h.completions).length, 0);
      const statHabits = document.getElementById("adminStatHabits");
      const statEntries = document.getElementById("adminStatEntries");
      const statQuotes = document.getElementById("adminStatQuotes");
      const statStorage = document.getElementById("adminStatStorage");

      if (statHabits) statHabits.textContent = `${appData.habits.length} / ${totalCheckins}`;
      if (statEntries) statEntries.textContent = `${appData.moods.length} / ${appData.diary.length} / ${appData.academics.assessments.length}`;
      if (statQuotes) statQuotes.textContent = appData.advice.length;

      const bytes = new Blob([localStorage.getItem(STORAGE_KEY) || ""]).size;
      if (statStorage) statStorage.textContent = `${(bytes / 1024).toFixed(1)} KB`;
      const bannerInput = document.getElementById("adminBannerInput");
      if (bannerInput) bannerInput.value = appData.announcement || "";

      const quotesEl = document.getElementById("adminQuotesList");
      if (quotesEl) {
        quotesEl.innerHTML = appData.advice.map((q, idx) => `
          <div class="admin-quote-row">
            <div style="flex:1;">
              <span class="badge">${escapeHtml(q.category)}</span>
              <p style="font-size:0.9rem; margin-top:0.25rem;">"${escapeHtml(q.text)}" — <strong>${escapeHtml(q.author || "Anonymous")}</strong></p>
            </div>
            <div class="action-group">
              <button type="button" class="btn-secondary small" onclick="adminPinSpotlight(${idx})">📌 Spotlight</button>
              <button type="button" class="icon-btn danger" onclick="adminDeleteQuote('${q.id}')">🗑</button>
            </div>
          </div>
        `).join("");
      }
    }

    window.adminPinSpotlight = (index) => {
      spotlightIndex = index;
      renderSpotlight();
      showToast("Quote pinned to Spotlight!");
    };

    window.adminDeleteQuote = (id) => {
      appData.advice = appData.advice.filter((q) => q.id !== id);
      saveData("Quote removed by Admin");
      renderAdvice();
      renderAdminDashboard();
    };

    function renderAll() {
      renderAnnouncementBanner();
      renderHeaderProfile();
      renderHomeHubSummary();
      renderHabits();
      renderAcademics();
      renderCalendar();
      renderAnalytics();
      renderMoods();
      renderDiary();
      renderAdvice();
      if (isAdminUnlocked) renderAdminDashboard();
    }

    function bindEvent(id, eventName, handler) {
      const el = document.getElementById(id);
      if (el) el.addEventListener(eventName, handler);
    }

    function openProfileSettings() {
      if (!appData.user) {
        appData.user = {
          uid: generateId("user"),
          name: "Guest User",
          email: "guest@gmail.com",
          photoURL: "https://ui-avatars.com/api/?name=Guest+User&background=4f46e5&color=fff",
          mode: "local"
        };
      }
      pendingEditAvatar = appData.user.photoURL;
      document.getElementById("editAvatarPreview").src = appData.user.photoURL;
      document.getElementById("editName").value = appData.user.name;
      document.getElementById("editEmail").value = appData.user.email;
      document.getElementById("firebaseConfigInput").value = localStorage.getItem(FIREBASE_CFG_KEY) || "";
      document.getElementById("profileModal")?.classList.remove("hidden");
    }

    document.addEventListener("DOMContentLoaded", () => {
      applyTheme(localStorage.getItem(THEME_KEY) || "light");
      initFirebaseIfAvailable();
      loadLocalData();
      startLiveClock();
      initAuth();

      const assessDateInput = document.getElementById("assessDate");
      if (assessDateInput) assessDateInput.value = formatLocalDateKey(new Date());
      renderAll();

      // Master delegated click handler for drawer, home tiles, back button, theme, profile & moods
      document.addEventListener("click", (e) => {
        if (e.target.closest("#menuToggleBtn") || e.target.closest("#sectionMenuBtn")) {
          openDrawer();
          return;
        }

        if (e.target.closest("#closeDrawerBtn") || e.target.closest("#drawerBackdrop")) {
          closeDrawer();
          return;
        }

        if (e.target.closest("#sectionBackBtn")) {
          switchView("home");
          return;
        }

        const navTargetEl = e.target.closest("[data-nav-target]");
        if (navTargetEl) {
          switchView(navTargetEl.dataset.navTarget);
          return;
        }

        if (e.target.closest("#darkModeToggle") || e.target.closest("#sectionDarkModeToggle")) {
          const nextTheme = document.body.classList.contains("dark-mode") ? "light" : "dark";
          applyTheme(nextTheme);
          return;
        }

        if (e.target.closest("#openProfileBtn") || e.target.closest("#sectionProfileBtn")) {
          openProfileSettings();
          return;
        }

        const moodBtn = e.target.closest(".mood-option");
        if (moodBtn) {
          document.querySelectorAll(".mood-option").forEach((b) => b.classList.remove("selected"));
          moodBtn.classList.add("selected");
          selectedMood = { mood: moodBtn.dataset.mood, emoji: moodBtn.dataset.emoji };
          const ind = document.getElementById("selectedMoodIndicator");
          if (ind) ind.textContent = `Selected: ${selectedMood.emoji} ${selectedMood.mood}`;
        }
      });

      // Dismiss Admin Banner
      bindEvent("dismissBannerBtn", "click", () => {
        document.getElementById("adminAnnouncementBar")?.classList.add("hidden");
      });

      // Calendar navigation & quick log buttons
      bindEvent("calPrevBtn", "click", () => {
        calendarViewDate.setMonth(calendarViewDate.getMonth() - 1);
        renderCalendar();
      });

      bindEvent("calNextBtn", "click", () => {
        calendarViewDate.setMonth(calendarViewDate.getMonth() + 1);
        renderCalendar();
      });

      bindEvent("calTodayBtn", "click", () => {
        calendarViewDate = new Date();
        calendarSelectedDateKey = formatLocalDateKey(new Date());
        renderCalendar();
      });

      bindEvent("quickLogMoodDateBtn", "click", () => {
        switchView("mood");
        showToast(`Logging mood for ${calendarSelectedDateKey}`);
      });

      bindEvent("quickLogDiaryDateBtn", "click", () => {
        switchView("diary");
        showToast(`Writing diary for ${calendarSelectedDateKey}`);
      });

      // Academics Forms, Search & Filters
      bindEvent("unitForm", "submit", (e) => {
        e.preventDefault();
        const code = document.getElementById("unitCode").value.trim().toUpperCase();
        const name = document.getElementById("unitName").value.trim();
        const semester = document.getElementById("unitSemester").value.trim() || "Sem 1";
        const target = Math.min(100, Math.max(1, Number(document.getElementById("unitTarget").value) || 70));
        const notes = document.getElementById("unitNotes").value.trim();
        if (!code || !name) return;

        appData.academics.units.unshift({
          id: generateId("unit"),
          code,
          name,
          semester,
          target,
          notes,
          createdAt: new Date().toISOString()
        });
        e.target.reset();
        document.getElementById("unitSemester").value = semester;
        document.getElementById("unitTarget").value = "70";
        saveData(`Added unit ${code}!`);
        renderAcademics();
      });

      bindEvent("assessmentForm", "submit", (e) => {
        e.preventDefault();
        const unitId = document.getElementById("assessUnitSelect").value;
        const type = document.getElementById("assessType").value;
        const topic = document.getElementById("assessTopic").value.trim();
        const score = parseFloat(document.getElementById("assessScore").value);
        const total = parseFloat(document.getElementById("assessTotal").value);
        const dateKey = document.getElementById("assessDate").value || formatLocalDateKey(new Date());
        const note = document.getElementById("assessNote").value.trim();

        if (!unitId) {
          showToast("Please add and select a Course Unit first!");
          return;
        }
        if (!topic || isNaN(score) || isNaN(total) || total <= 0 || score < 0) {
          showToast("Enter valid marks (e.g., 24 out of 30).");
          return;
        }

        appData.academics.assessments.unshift({
          id: generateId("assess"),
          unitId,
          type,
          topic,
          score,
          total,
          dateKey,
          note,
          createdAt: new Date().toISOString()
        });
        e.target.reset();
        document.getElementById("assessUnitSelect").value = unitId;
        document.getElementById("assessDate").value = formatLocalDateKey(new Date());
        const pct = Math.round((score / total) * 100);
        saveData(`Recorded ${type}: ${pct}% (${getGradeLetter(pct)})!`);
        renderAcademics();
        renderCalendar();
      });

      bindEvent("acadSearchInput", "input", renderAcademics);
      bindEvent("acadFilterUnit", "change", renderAcademics);
      bindEvent("acadFilterType", "change", renderAcademics);

      // Universal Edit Entry Modal controls
      const closeEditModal = () => {
        document.getElementById("editEntryModal")?.classList.add("hidden");
        editingContext = null;
      };
      bindEvent("closeEditModal", "click", closeEditModal);
      bindEvent("cancelEditModalBtn", "click", closeEditModal);

      bindEvent("editEntryForm", "submit", (e) => {
        e.preventDefault();
        if (!editingContext) return;
        const { type, id } = editingContext;

        if (type === "habit") {
          const h = appData.habits.find((x) => x.id === id);
          if (h) {
            h.title = document.getElementById("editHabitTitle").value.trim();
            h.category = document.getElementById("editHabitCat").value;
            saveData("Habit updated!");
            renderHabits();
            renderCalendar();
            renderAnalytics();
          }
        } else if (type === "mood") {
          const m = appData.moods.find((x) => x.id === id);
          if (m) {
            const [moodName, moodEmoji] = document.getElementById("editMoodSelect").value.split("|");
            m.mood = moodName;
            m.emoji = moodEmoji;
            m.note = document.getElementById("editMoodNote").value.trim();
            saveData("Mood updated!");
            renderMoods();
            renderCalendar();
            renderAnalytics();
          }
        } else if (type === "diary") {
          const d = appData.diary.find((x) => x.id === id);
          if (d) {
            d.title = document.getElementById("editDiaryTitle").value.trim();
            d.tag = document.getElementById("editDiaryTag").value;
            d.content = document.getElementById("editDiaryContent").value.trim();
            saveData("Diary entry updated!");
            renderDiary();
            renderCalendar();
          }
        } else if (type === "unit") {
          const u = appData.academics.units.find((x) => x.id === id);
          if (u) {
            u.code = document.getElementById("editUnitCode").value.trim().toUpperCase();
            u.name = document.getElementById("editUnitName").value.trim();
            u.semester = document.getElementById("editUnitSemester").value.trim() || "Sem 1";
            u.target = Math.min(100, Math.max(1, Number(document.getElementById("editUnitTarget").value) || 70));
            u.notes = document.getElementById("editUnitNotes").value.trim();
            saveData("Course unit updated!");
            renderAcademics();
          }
        } else if (type === "assessment") {
          const a = appData.academics.assessments.find((x) => x.id === id);
          if (a) {
            a.type = document.getElementById("editAssessType").value;
            a.topic = document.getElementById("editAssessTopic").value.trim();
            a.score = parseFloat(document.getElementById("editAssessScore").value) || 0;
            a.total = Math.max(1, parseFloat(document.getElementById("editAssessTotal").value) || 1);
            a.dateKey = document.getElementById("editAssessDate").value || a.dateKey;
            a.note = document.getElementById("editAssessNote").value.trim();
            saveData("Result updated!");
            renderAcademics();
            renderCalendar();
          }
        }

        closeEditModal();
      });

      // Admin Unlock, Lock, Banner, PIN & Reset
      bindEvent("adminUnlockForm", "submit", (e) => {
        e.preventDefault();
        const entered = document.getElementById("adminPinInput").value.trim();
        if (entered === (appData.adminPin || "1234")) {
          isAdminUnlocked = true;
          document.getElementById("adminPinInput").value = "";
          document.getElementById("adminLockScreen").classList.add("hidden");
          document.getElementById("adminDashboardContent").classList.remove("hidden");
          document.getElementById("adminLockBtn").classList.remove("hidden");
          renderAdminDashboard();
          showToast("Admin Dashboard unlocked!");
        } else {
          showToast("Incorrect Admin PIN.");
        }
      });

      bindEvent("adminLockBtn", "click", () => {
        isAdminUnlocked = false;
        document.getElementById("adminLockScreen").classList.remove("hidden");
        document.getElementById("adminDashboardContent").classList.add("hidden");
        document.getElementById("adminLockBtn").classList.add("hidden");
        showToast("Admin panel locked.");
      });

      bindEvent("saveAdminBannerBtn", "click", () => {
        appData.announcement = document.getElementById("adminBannerInput").value.trim();
        renderAnnouncementBanner();
        saveData("Announcement banner published!");
      });

      bindEvent("clearAdminBannerBtn", "click", () => {
        appData.announcement = "";
        document.getElementById("adminBannerInput").value = "";
        renderAnnouncementBanner();
        saveData("Announcement banner cleared.");
      });

      bindEvent("changeAdminPinBtn", "click", () => {
        const newPin = document.getElementById("newAdminPinInput").value.trim();
        if (newPin.length < 4) {
          showToast("PIN must be at least 4 characters.");
          return;
        }
        appData.adminPin = newPin;
        document.getElementById("newAdminPinInput").value = "";
        saveData("Admin PIN updated!");
      });

      bindEvent("adminLoadDemoBtn", "click", () => {
        const today = formatLocalDateKey(new Date());
        const uId = generateId("unit");
        appData.academics.units.unshift({
          id: uId,
          code: "EME 2101",
          name: "Engineering Mechanics",
          semester: "Y2 Sem 1",
          target: 75,
          notes: "Focus on trusses and bending moments"
        });
        appData.academics.assessments.unshift(
          { id: generateId("assess"), unitId: uId, type: "CAT 1", topic: "Equilibrium of Rigid Bodies", score: 24, total: 30, dateKey: today, note: "Strong on free body diagrams", createdAt: new Date().toISOString() },
          { id: generateId("assess"), unitId: uId, type: "Topical Revision", topic: "Shear Force & Bending Moment", score: 11, total: 20, dateKey: today, note: "Review distributed loads", createdAt: new Date().toISOString() }
        );
        appData.habits.unshift({
          id: generateId("hab"),
          title: "Revise 5 topical engineering questions",
          category: "Learning",
          createdAt: new Date().toISOString(),
          completions: { [today]: new Date().toISOString() }
        });
        saveData("Sample academic & habit data loaded!");
        renderAll();
      });

      bindEvent("adminExportAllBtn", "click", () => {
        document.getElementById("exportBackupBtn")?.click();
      });

      bindEvent("adminResetDataBtn", "click", () => {
        const btn = document.getElementById("adminResetDataBtn");
        if (btn.dataset.armed !== "true") {
          btn.dataset.armed = "true";
          btn.textContent = "⚠️ Tap Again to Confirm Wipe";
          setTimeout(() => {
            btn.dataset.armed = "false";
            btn.textContent = "⚠️ Reset All App Data";
          }, 4000);
          return;
        }
        localStorage.removeItem(STORAGE_KEY);
        showToast("All data wiped. Reloading...");
        setTimeout(() => location.reload(), 800);
      });

      // Avatar upload on signup
      bindEvent("authAvatarInput", "change", (e) => {
        if (e.target.files[0]) compressImageFile(e.target.files[0], (url) => {
          pendingAuthAvatar = url;
          document.getElementById("authAvatarPreview").src = url;
        });
      });

      // Google Sign-In handler
      bindEvent("googleSignInBtn", "click", async () => {
        if (isFirebaseConfigured && auth) {
          try {
            await auth.signInWithPopup(new firebase.auth.GoogleAuthProvider());
            showToast("Signed in with Google!");
          } catch (err) { showToast("Google Sign-In error: " + err.message); }
        } else {
          document.getElementById("authName")?.focus();
          showToast("Enter your Name & Gmail below to start immediately!");
        }
      });

      // Local profile creation
      bindEvent("localAccountForm", "submit", (e) => {
        e.preventDefault();
        const name = document.getElementById("authName").value.trim();
        const email = document.getElementById("authEmail").value.trim();
        const photoURL = pendingAuthAvatar || `https://ui-avatars.com/api/?name=${encodeURIComponent(name)}&background=4f46e5&color=fff`;
        appData.user = { uid: generateId("user"), name, email, photoURL, mode: "local" };
        document.getElementById("authOverlay").classList.add("hidden");
        saveData(`Welcome, ${name}!`);
        renderAll();
      });

      // Profile Modal controls
      bindEvent("closeProfileModal", "click", () => {
        document.getElementById("profileModal")?.classList.add("hidden");
      });

      bindEvent("editAvatarInput", "change", (e) => {
        if (e.target.files[0]) compressImageFile(e.target.files[0], (url) => {
          pendingEditAvatar = url;
          document.getElementById("editAvatarPreview").src = url;
        });
      });

      bindEvent("editProfileForm", "submit", (e) => {
        e.preventDefault();
        appData.user.name = document.getElementById("editName").value.trim();
        appData.user.email = document.getElementById("editEmail").value.trim();
        if (pendingEditAvatar) appData.user.photoURL = pendingEditAvatar;
        document.getElementById("profileModal")?.classList.add("hidden");
        saveData("Profile updated!");
        renderHeaderProfile();
      });

      bindEvent("signOutBtn", "click", async () => {
        if (auth && appData.user?.mode === "google") await auth.signOut();
        appData.user = null;
        localStorage.setItem(STORAGE_KEY, JSON.stringify(appData));
        renderHeaderProfile();
        document.getElementById("profileModal")?.classList.add("hidden");
        document.getElementById("authOverlay")?.classList.remove("hidden");
      });

      // JSON Backup Export & Import
      bindEvent("exportBackupBtn", "click", () => {
        const blob = new Blob([JSON.stringify(appData, null, 2)], { type: "application/json" });
        const url = URL.createObjectURL(blob);
        const a = document.createElement("a");
        a.href = url;
        a.download = `lifepulse-backup-${formatLocalDateKey(new Date())}.json`;
        a.click();
        URL.revokeObjectURL(url);
        showToast("Backup downloaded!");
      });

      bindEvent("importBackupInput", "change", (e) => {
        const file = e.target.files[0];
        if (!file) return;
        const reader = new FileReader();
        reader.onload = (ev) => {
          try {
            const parsed = JSON.parse(ev.target.result);
            appData = {
              ...appData,
              ...parsed,
              academics: {
                units: parsed.academics?.units || [],
                assessments: parsed.academics?.assessments || []
              }
            };
            saveData("Backup restored!");
            renderAll();
            document.getElementById("profileModal")?.classList.add("hidden");
          } catch { showToast("Invalid backup file."); }
        };
        reader.readAsText(file);
      });

      // Firebase JSON config form
      bindEvent("firebaseConfigForm", "submit", (e) => {
        e.preventDefault();
        const raw = document.getElementById("firebaseConfigInput").value.trim();
        if (!raw) {
          localStorage.removeItem(FIREBASE_CFG_KEY);
          showToast("Firebase config cleared. Reloading...");
          setTimeout(() => location.reload(), 800);
          return;
        }
        try {
          const parsed = JSON.parse(raw);
          if (!parsed.apiKey || !parsed.projectId) throw new Error("Missing apiKey or projectId");
          localStorage.setItem(FIREBASE_CFG_KEY, JSON.stringify(parsed));
          showToast("Firebase saved! Reloading...");
          setTimeout(() => location.reload(), 800);
        } catch (err) {
          showToast("Please paste valid JSON with double quotes around keys.");
        }
      });

      // Habits events
      bindEvent("habitDatePicker", "change", (e) => {
        selectedDateKey = e.target.value || formatLocalDateKey(new Date());
        renderHabits();
      });

      bindEvent("habitForm", "submit", (e) => {
        e.preventDefault();
        const title = document.getElementById("habitTitle").value.trim();
        const category = document.getElementById("habitCategory").value;
        if (!title) return;
        appData.habits.unshift({ id: generateId("hab"), title, category, createdAt: new Date().toISOString(), completions: {} });
        e.target.reset();
        saveData("New habit added!");
        renderHabits();
        renderCalendar();
        renderAnalytics();
      });

      // Mood save button
      bindEvent("saveMoodBtn", "click", () => {
        if (!selectedMood) { showToast("Please pick a mood emoji first!"); return; }
        const note = document.getElementById("moodNote").value.trim();
        appData.moods.unshift({
          id: generateId("mood"), mood: selectedMood.mood, emoji: selectedMood.emoji, note,
          dateKey: calendarSelectedDateKey || formatLocalDateKey(new Date()), createdAt: new Date().toISOString()
        });
        document.getElementById("moodNote").value = "";
        document.querySelectorAll(".mood-option").forEach((b) => b.classList.remove("selected"));
        selectedMood = null;
        document.getElementById("selectedMoodIndicator").textContent = "No mood selected yet";
        saveData("Mood saved!");
        renderMoods();
        renderCalendar();
        renderAnalytics();
      });

      // Diary events
      bindEvent("diaryForm", "submit", (e) => {
        e.preventDefault();
        const title = document.getElementById("diaryTitle").value.trim();
        const tag = document.getElementById("diaryTag").value;
        const content = document.getElementById("diaryContent").value.trim();
        if (!title || !content) return;
        appData.diary.unshift({
          id: generateId("diary"), title, tag, content,
          dateKey: calendarSelectedDateKey || formatLocalDateKey(new Date()), createdAt: new Date().toISOString()
        });
        e.target.reset();
        saveData("Diary entry saved!");
        renderDiary();
        renderCalendar();
      });

      bindEvent("diarySearch", "input", renderDiary);

      // Advice & Quotes events
      bindEvent("nextSpotlightBtn", "click", () => {
        spotlightIndex++;
        renderSpotlight();
      });

      bindEvent("adviceForm", "submit", (e) => {
        e.preventDefault();
        const text = document.getElementById("adviceText").value.trim();
        const author = document.getElementById("adviceAuthor").value.trim() || appData.user?.name || "Anonymous";
        const category = document.getElementById("adviceCategory").value;
        if (!text) return;
        appData.advice.unshift({
          id: generateId("adv"), text, author, category, likes: 0, liked: false, createdAt: new Date().toISOString()
        });
        e.target.reset();
        saveData("Quote uploaded!");
        renderAdvice();
      });

      bindEvent("adviceFilterCategory", "change", renderAdvice);
    });
  </script>
</body>
</html>
```# Life-cycle