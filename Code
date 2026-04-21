"""
OneKey Switch v2 (упрощённая версия на keyboard)
Запуск обязательно от имени администратора!
"""

import keyboard
import threading
import time
import json
import os
import sys
import ctypes
import tkinter as tk

# ── Авто-перезапуск с правами ────────────────────────────────────────────────
def is_admin():
    try:
        return ctypes.windll.shell32.IsUserAnAdmin()
    except:
        return False

if not is_admin():
    ctypes.windll.shell32.ShellExecuteW(None, "runas", sys.executable, " ".join(sys.argv), None, 1)
    sys.exit()

# ── Конфиг ────────────────────────────────────────────────────────────────────
CONFIG_FILE = os.path.join(os.path.dirname(os.path.abspath(__file__)), "onekey.json")

DEFAULT_CONFIG = {
    "key_ru":         "F1",
    "key_en":         "F2",
    "key_copy":       "F3",
    "key_paste":      "F4",
    "popup_x":        50,
    "popup_y":        50,
    "popup_preset":   "center",
    "popup_duration": 1500,
    "follow_cursor":  False,
    "popup_size":     32,
    "theme":          "dark",
    "lang":           "ru",
}

PRESETS = {
    "center":        (50, 50),
    "top_right":     (95,  5),
    "bottom_right":  (95, 95),
    "top_left":      ( 5,  5),
    "bottom_left":   ( 5, 95),
    "top_center":    (50,  5),
    "bottom_center": (50, 95),
}

PRESET_GRID = [
    ["top_left",    "top_center",    "top_right"   ],
    [None,          "center",        None          ],
    ["bottom_left", "bottom_center", "bottom_right"],
]

def load_config():
    if os.path.exists(CONFIG_FILE):
        try:
            with open(CONFIG_FILE, "r", encoding="utf-8") as f:
                cfg = json.load(f)
            for k, v in DEFAULT_CONFIG.items():
                cfg.setdefault(k, v)
            return cfg
        except:
            pass
    return DEFAULT_CONFIG.copy()

def save_config(cfg):
    with open(CONFIG_FILE, "w", encoding="utf-8") as f:
        json.dump(cfg, f, indent=2, ensure_ascii=False)

config = load_config()

# ── Локализация ───────────────────────────────────────────────────────────────
I18N = {
    "ru": {
        "title":          "OneKey Switch",
        "tab_keys":       "Клавиши",
        "tab_popup":      "Popup",
        "tab_settings":   "Настройки",
        "lbl_ru":         "Русский язык",
        "lbl_en":         "Английский язык",
        "lbl_copy":       "Копировать  Ctrl+C",
        "lbl_paste":      "Вставить    Ctrl+V",
        "click_key":      "Нажмите кнопку, затем нужную клавишу",
        "popup_pos":      "Положение popup",
        "manual_xy":      "Вручную X / Y",
        "follow_cursor":  "Следовать за курсором",
        "duration":       "Длительность popup (мс)",
        "popup_size_lbl": "Размер шрифта popup",
        "theme_lbl":      "Тема",
        "lang_lbl":       "Язык интерфейса",
        "save":           "Сохранить и применить",
        "status":         "● АКТИВЕН",
        "presets": {
            "center":        "Центр",
            "top_right":     "Пр.верх",
            "bottom_right":  "Пр.низ",
            "top_left":      "Лв.верх",
            "bottom_left":   "Лв.низ",
            "top_center":    "Верх",
            "bottom_center": "Низ",
        },
        "themes": {"dark": "Тёмная", "light": "Светлая", "blue": "Голубая"},
        "langs":  {"ru": "Русский", "en": "English"},
    },
    "en": {
        "title":          "OneKey Switch",
        "tab_keys":       "Keys",
        "tab_popup":      "Popup",
        "tab_settings":   "Settings",
        "lbl_ru":         "Russian layout",
        "lbl_en":         "English layout",
        "lbl_copy":       "Copy        Ctrl+C",
        "lbl_paste":      "Paste       Ctrl+V",
        "click_key":      "Click a button, then press the desired key",
        "popup_pos":      "Popup position",
        "manual_xy":      "Manual X / Y",
        "follow_cursor":  "Follow cursor",
        "duration":       "Popup duration (ms)",
        "popup_size_lbl": "Popup font size",
        "theme_lbl":      "Theme",
        "lang_lbl":       "Interface language",
        "save":           "Save & Apply",
        "status":         "● ACTIVE",
        "presets": {
            "center":        "Center",
            "top_right":     "TR",
            "bottom_right":  "BR",
            "top_left":      "TL",
            "bottom_left":   "BL",
            "top_center":    "Top",
            "bottom_center": "Bot",
        },
        "themes": {"dark": "Dark", "light": "Light", "blue": "Blue"},
        "langs":  {"ru": "Русский", "en": "English"},
    },
}

def t(key, sub=None):
    d = I18N.get(config.get("lang", "ru"), I18N["ru"])
    if sub:
        return d.get(sub, {}).get(key, key)
    return d.get(key, key)

# ── Темы ──────────────────────────────────────────────────────────────────────
THEMES = {
    "dark": dict(
        bg="#0f0f0f", surface="#1a1a1a", surf2="#252525",
        text="#e8e8e8", text2="#666666", acc="#ffffff",
        green="#4caf50", btn_fg="#000000", btn_bg="#ffffff",
        popup_bg="#0d0d0d", popup_fg="#ffffff",
    ),
    "light": dict(
        bg="#f2f2f2", surface="#ffffff", surf2="#e4e4e4",
        text="#111111", text2="#888888", acc="#111111",
        green="#2e7d32", btn_fg="#ffffff", btn_bg="#111111",
        popup_bg="#ffffff", popup_fg="#111111",
    ),
    "blue": dict(
        bg="#ddeeff", surface="#ffffff", surf2="#c2ddf5",
        text="#0d3b5e", text2="#5a8aaa", acc="#0d3b5e",
        green="#1565c0", btn_fg="#ffffff", btn_bg="#1565c0",
        popup_bg="#1565c0", popup_fg="#ffffff",
    ),
}

def th(key):
    return THEMES.get(config.get("theme", "dark"), THEMES["dark"])[key]

# ── Захват клавиши для GUI ────────────────────────────────────────────────────
_capture_callback = None

def start_capture(callback):
    global _capture_callback
    _capture_callback = callback

def stop_capture():
    global _capture_callback
    _capture_callback = None


# ── Popup ─────────────────────────────────────────────────────────────────────
popup_win = None
popup_timer = None

def show_popup(text):
    global popup_win, popup_timer
    if popup_timer:
        popup_timer.cancel()

    def _create():
        global popup_win, popup_timer
        if popup_win and popup_win.winfo_exists():
            popup_win.destroy()

        win = tk.Toplevel(root)
        win.overrideredirect(True)
        win.attributes("-topmost", True)
        win.attributes("-alpha", 0.0)

        lbl = tk.Label(win, text=text, font=("Consolas", config.get("popup_size", 32), "bold"),
                       fg=th("popup_fg"), bg=th("popup_bg"), padx=26, pady=16)
        lbl.pack()
        win.update_idletasks()

        w = win.winfo_width()
        h = win.winfo_height()
        sw = win.winfo_screenwidth()
        sh = win.winfo_screenheight()

        if config.get("follow_cursor"):
            x = win.winfo_pointerx() - w // 2
            y = win.winfo_pointery() - h - 20
        else:
            x = int((sw - w) * config.get("popup_x", 50) / 100)
            y = int((sh - h) * config.get("popup_y", 50) / 100)

        x = max(0, min(x, sw - w))
        y = max(0, min(y, sh - h))
        win.geometry(f"+{x}+{y}")
        popup_win = win

        def fade_in(a=0.0):
            if popup_win and popup_win.winfo_exists():
                popup_win.attributes("-alpha", min(a, 1.0))
                if a < 1.0:
                    popup_win.after(16, lambda: fade_in(a + 0.14))
        fade_in()

        dur = config.get("popup_duration", 1500) / 1000
        popup_timer = threading.Timer(dur, lambda: root.after(0, hide_popup))
        popup_timer.start()

    root.after(0, _create)

def hide_popup():
    global popup_win
    if not popup_win or not popup_win.winfo_exists():
        return
    def fade_out(a=1.0):
        if popup_win and popup_win.winfo_exists():
            popup_win.attributes("-alpha", max(a, 0.0))
            if a > 0:
                popup_win.after(16, lambda: fade_out(a - 0.16))
            else:
                popup_win.destroy()
    fade_out()

# ── Действия (как в рабочей версии) ───────────────────────────────────────────
def switch_to_ru():
    keyboard.send("alt+shift")
    show_popup("RU")

def switch_to_en():
    keyboard.send("alt+shift")
    show_popup("ENG")

def do_copy():
    keyboard.send("ctrl+c")
    show_popup("COPY")

def do_paste():
    keyboard.send("ctrl+v")
    show_popup("PASTE")

# ── Хоткеи ────────────────────────────────────────────────────────────────────
hotkey_handles = []

def rebind_hotkeys():
    global hotkey_handles
    for h in hotkey_handles:
        try:
            keyboard.remove_hotkey(h)
        except:
            pass
    hotkey_handles.clear()

    def add(k, fn):
        def wrapped():
            if _capture_callback is not None:
                # В режиме захвата — перехватываем имя клавиши через keyboard
                # (keyboard уже вернул имя через хоткей, значит клавиша = k)
                cb = _capture_callback
                stop_capture()
                root.after(0, lambda name=k: cb(name))
                return
            fn()
        try:
            h = keyboard.add_hotkey(k, wrapped, suppress=True)
            hotkey_handles.append(h)
        except:
            pass

    add(config["key_ru"],   switch_to_ru)
    add(config["key_en"],   switch_to_en)
    add(config["key_copy"], do_copy)
    add(config["key_paste"], do_paste)

# ── GUI ───────────────────────────────────────────────────────────────────────
root = None
settings_win = None

def open_settings():
    global settings_win
    if settings_win and settings_win.winfo_exists():
        settings_win.lift()
        return
    settings_win = tk.Toplevel(root)
    _build_settings(settings_win)

def _build_settings(win):
    win.title(t("title"))
    win.configure(bg=th("bg"))
    win.resizable(False, False)
    sw_px, sh_px = win.winfo_screenwidth(), win.winfo_screenheight()
    W, H = 530, 640
    win.geometry(f"{W}x{H}+{(sw_px-W)//2}+{(sh_px-H)//2}")

    # Titlebar
    hdr = tk.Frame(win, bg=th("surface"), pady=10)
    hdr.pack(fill="x")
    tk.Label(hdr, text="ONEKEY SWITCH",
             font=("Consolas", 12, "bold"), fg=th("text"), bg=th("surface")
             ).pack(side="left", padx=16)
    tk.Label(hdr, text=t("status"),
             font=("Consolas", 9), fg=th("green"), bg=th("surface")
             ).pack(side="right", padx=16)

    # Tabs
    tab_bar   = tk.Frame(win, bg=th("surface"))
    tab_bar.pack(fill="x")
    container = tk.Frame(win, bg=th("bg"))
    container.pack(fill="both", expand=True)

    panels   = {}
    tab_btns = {}

    def show_tab(name):
        for p in panels.values():
            p.pack_forget()
        panels[name].pack(fill="both", expand=True, padx=16, pady=12)
        for n, b in tab_btns.items():
            active = (n == name)
            b.config(bg=th("surface") if active else th("bg"),
                     fg=th("text")    if active else th("text2"))

    for tab_key in ("tab_keys", "tab_popup", "tab_settings"):
        p = tk.Frame(container, bg=th("bg"))
        panels[tab_key] = p
        b = tk.Button(tab_bar, text=t(tab_key),
                      font=("Consolas", 10), relief="flat",
                      bg=th("bg"), fg=th("text2"),
                      activebackground=th("surface"),
                      cursor="hand2", padx=14, pady=8,
                      command=lambda n=tab_key: show_tab(n))
        b.pack(side="left")
        tab_btns[tab_key] = b

    # ─ TAB: Клавиши ──────────────────────────────────────────────────────────
    p = panels["tab_keys"]
    actions = [
        ("key_ru",    "lbl_ru"),
        ("key_en",    "lbl_en"),
        ("key_copy",  "lbl_copy"),
        ("key_paste", "lbl_paste"),
    ]
    key_vars = {}

    for cfg_key, lbl_key in actions:
        row = tk.Frame(p, bg=th("surface"), pady=8, padx=12)
        row.pack(fill="x", pady=3)
        tk.Label(row, text=t(lbl_key),
                 font=("Consolas", 10), fg=th("text"), bg=th("surface")
                 ).pack(side="left")

        var = tk.StringVar(value=config[cfg_key])
        key_vars[cfg_key] = var

        btn_ref = [None]
        btn = tk.Button(row, textvariable=var,
                        font=("Consolas", 10, "bold"),
                        fg=th("acc"), bg=th("surf2"),
                        relief="flat", cursor="hand2", width=8,
                        activebackground=th("surface"))
        btn.pack(side="right")
        btn_ref[0] = btn

        def make_click(ck=cfg_key, v=var, br=btn_ref):
            def click():
                b = br[0]
                if _capture_callback is not None:
                    stop_capture()
                    v.set(config[ck])
                    b.config(bg=th("surf2"))
                    return
                v.set("...")
                b.config(bg="#1c1800" if config.get("theme") == "dark" else "#fff9c4")

                def on_captured(name):
                    v.set(name)
                    config[ck] = name
                    b.config(bg=th("surf2"))
                    stop_capture()
                    rebind_hotkeys()
                start_capture(on_captured)
            return click

        btn.config(command=make_click())

    tk.Label(p, text=t("click_key"), font=("Consolas", 9),
             fg=th("text2"), bg=th("bg")).pack(anchor="w", pady=(8, 0))

    # ─ TAB: Popup ─────────────────────────────────────────────────────────────
    p = panels["tab_popup"]

    tk.Label(p, text=t("popup_pos"), font=("Consolas", 9),
             fg=th("text2"), bg=th("bg")).pack(anchor="w", pady=(0, 6))

    preset_var = tk.StringVar(value=config.get("popup_preset", "center"))
    x_var      = tk.IntVar(value=config.get("popup_x", 50))
    y_var      = tk.IntVar(value=config.get("popup_y", 50))

    grid_frame = tk.Frame(p, bg=th("surface"), pady=8, padx=10)
    grid_frame.pack(fill="x", pady=3)
    preset_btns = {}

    def select_preset(pk):
        preset_var.set(pk)
        if pk in PRESETS:
            x_var.set(PRESETS[pk][0])
            y_var.set(PRESETS[pk][1])
        for k, b in preset_btns.items():
            active = (k == pk)
            b.config(bg=th("btn_bg") if active else th("surf2"),
                     fg=th("btn_fg") if active else th("text"))

    for ri, row_keys in enumerate(PRESET_GRID):
        for ci, pk in enumerate(row_keys):
            if pk is None:
                tk.Frame(grid_frame, bg=th("surface"), width=54, height=36
                         ).grid(row=ri, column=ci, padx=3, pady=3)
                continue
            active = (pk == preset_var.get())
            b = tk.Button(grid_frame, text=t(pk, "presets"),
                          font=("Consolas", 8), width=7, height=2,
                          bg=th("btn_bg") if active else th("surf2"),
                          fg=th("btn_fg") if active else th("text"),
                          relief="flat", cursor="hand2",
                          command=lambda k=pk: select_preset(k))
            b.grid(row=ri, column=ci, padx=3, pady=3)
            preset_btns[pk] = b

    # X/Y ручной
    xy_frame = tk.Frame(p, bg=th("surface"), pady=8, padx=12)
    xy_frame.pack(fill="x", pady=(8, 3))
    tk.Label(xy_frame, text=t("manual_xy"), font=("Consolas", 9),
             fg=th("text2"), bg=th("surface")).pack(anchor="w", pady=(0, 4))

    for lbl, var in [("X:", x_var), ("Y:", y_var)]:
        r = tk.Frame(xy_frame, bg=th("surface"))
        r.pack(fill="x", pady=2)
        tk.Label(r, text=lbl, font=("Consolas", 10),
                 fg=th("text"), bg=th("surface"), width=3).pack(side="left")
        val_lbl = tk.Label(r, text=str(var.get()),
                           font=("Consolas", 10, "bold"),
                           fg=th("acc"), bg=th("surface"), width=4)
        val_lbl.pack(side="right")
        tk.Scale(r, variable=var, from_=0, to=100, orient="horizontal",
                 bg=th("surface"), fg=th("text2"),
                 troughcolor=th("surf2"), highlightthickness=0, sliderrelief="flat",
                 command=lambda v, lbl2=val_lbl: lbl2.config(text=str(int(float(v))))
                 ).pack(side="left", fill="x", expand=True)

    # Follow cursor
    cursor_var = tk.BooleanVar(value=config.get("follow_cursor", False))
    c_row = tk.Frame(p, bg=th("surface"), pady=8, padx=12)
    c_row.pack(fill="x", pady=3)
    tk.Checkbutton(c_row, text=t("follow_cursor"), variable=cursor_var,
                   font=("Consolas", 10), fg=th("text"), bg=th("surface"),
                   selectcolor=th("surf2"), activebackground=th("surface"),
                   relief="flat").pack(anchor="w")

    # Длительность
    dur_frame = tk.Frame(p, bg=th("surface"), pady=8, padx=12)
    dur_frame.pack(fill="x", pady=3)
    tk.Label(dur_frame, text=t("duration"), font=("Consolas", 9),
             fg=th("text2"), bg=th("surface")).pack(anchor="w", pady=(0, 4))
    dur_var = tk.IntVar(value=config.get("popup_duration", 1500))
    dur_lbl = tk.Label(dur_frame, text=f"{dur_var.get()} ms",
                       font=("Consolas", 10, "bold"),
                       fg=th("acc"), bg=th("surface"), width=7)
    dur_lbl.pack(side="right")
    tk.Scale(dur_frame, variable=dur_var, from_=500, to=5000, resolution=100,
             orient="horizontal", bg=th("surface"), fg=th("text2"),
             troughcolor=th("surf2"), highlightthickness=0, sliderrelief="flat",
             command=lambda v: dur_lbl.config(text=f"{int(float(v))} ms")
             ).pack(side="left", fill="x", expand=True)

    # ─ TAB: Настройки ─────────────────────────────────────────────────────────
    p = panels["tab_settings"]

    def make_radio_row(label_key, variable, options_dict):
        row = tk.Frame(p, bg=th("surface"), pady=8, padx=12)
        row.pack(fill="x", pady=3)
        tk.Label(row, text=t(label_key), font=("Consolas", 10),
                 fg=th("text"), bg=th("surface"), width=22, anchor="w"
                 ).pack(side="left")
        for val, lbl in options_dict.items():
            tk.Radiobutton(row, text=lbl, variable=variable, value=val,
                           font=("Consolas", 10), fg=th("text"), bg=th("surface"),
                           selectcolor=th("surf2"), activebackground=th("surface"),
                           relief="flat").pack(side="left", padx=6)
        return row

    theme_var = tk.StringVar(value=config.get("theme", "dark"))
    lang_var  = tk.StringVar(value=config.get("lang",  "ru"))

    make_radio_row("theme_lbl", theme_var, t("themes"))
    make_radio_row("lang_lbl",  lang_var,  t("langs"))

    # Размер popup
    sz_row = tk.Frame(p, bg=th("surface"), pady=8, padx=12)
    sz_row.pack(fill="x", pady=3)
    tk.Label(sz_row, text=t("popup_size_lbl"), font=("Consolas", 10),
             fg=th("text"), bg=th("surface")).pack(side="left")
    size_var = tk.IntVar(value=config.get("popup_size", 32))
    size_lbl = tk.Label(sz_row, text=f"{size_var.get()} px",
                        font=("Consolas", 10, "bold"),
                        fg=th("acc"), bg=th("surface"), width=6)
    size_lbl.pack(side="right")
    tk.Scale(sz_row, variable=size_var, from_=16, to=72, resolution=2,
             orient="horizontal", bg=th("surface"), fg=th("text2"),
             troughcolor=th("surf2"), highlightthickness=0, sliderrelief="flat",
             command=lambda v: size_lbl.config(text=f"{int(float(v))} px")
             ).pack(side="left", fill="x", expand=True)

    # ─ Кнопка Сохранить ───────────────────────────────────────────────────────
    def save_and_apply():
        for ck in ("key_ru", "key_en", "key_copy", "key_paste"):
            config[ck] = key_vars[ck].get()
        config["popup_x"]        = x_var.get()
        config["popup_y"]        = y_var.get()
        config["popup_preset"]   = preset_var.get()
        config["popup_duration"] = dur_var.get()
        config["follow_cursor"]  = cursor_var.get()
        config["theme"]          = theme_var.get()
        config["lang"]           = lang_var.get()
        config["popup_size"]     = size_var.get()
        save_config(config)
        rebind_hotkeys()
        show_popup("OK")
        win.after(450, win.destroy)

    footer = tk.Frame(win, bg=th("bg"), pady=10, padx=16)
    footer.pack(fill="x", side="bottom")
    tk.Button(footer, text=t("save"),
              font=("Consolas", 10, "bold"),
              fg=th("btn_fg"), bg=th("btn_bg"),
              relief="flat", cursor="hand2",
              activebackground=th("surf2"),
              pady=8, command=save_and_apply
              ).pack(fill="x")

    show_tab("tab_keys")

# ── Иконка трея ───────────────────────────────────────────────────────────────
def _make_tray_icon():
    try:
        from PIL import Image, ImageDraw, ImageFont
    except ImportError:
        return None
    size = 64
    img  = Image.new("RGBA", (size, size), (0, 0, 0, 0))
    d    = ImageDraw.Draw(img)
    d.rounded_rectangle([2, 2, size-3, size-3], radius=12,
                        fill="#0d0d0d", outline="#ffffff", width=2)
    try:
        fnt = ImageFont.truetype("consola.ttf", 22)
    except Exception:
        fnt = ImageFont.load_default()
    text = "1K"
    bbox = d.textbbox((0, 0), text, font=fnt)
    tw = bbox[2] - bbox[0]
    th2 = bbox[3] - bbox[1]
    d.text(((size - tw) // 2 - bbox[0], (size - th2) // 2 - bbox[1] + 1),
           text, font=fnt, fill="#ffffff")
    return img

def start_tray():
    try:
        import pystray
    except ImportError:
        print("[трей] pystray не найден. pip install pystray pillow")
        return

    img = _make_tray_icon()
    if img is None:
        print("[трей] pillow не найден. pip install pillow")
        return

    def on_settings(icon, item):
        root.after(0, open_settings)

    def on_quit(icon, item):
        icon.stop()
        for h in hotkey_handles:
            try:
                keyboard.remove_hotkey(h)
            except Exception:
                pass
        root.after(0, root.quit)

    icon = pystray.Icon(
        "onekey_switch", img, "OneKey Switch",
        menu=pystray.Menu(
            pystray.MenuItem(
                "OneKey Switch  ●  активен", None, enabled=False),
            pystray.Menu.SEPARATOR,
            pystray.MenuItem(
                lambda item: f"RU  ←  {config['key_ru']}",
                lambda icon, item: root.after(0, lambda: show_popup("RU"))),
            pystray.MenuItem(
                lambda item: f"ENG ←  {config['key_en']}",
                lambda icon, item: root.after(0, lambda: show_popup("ENG"))),
            pystray.Menu.SEPARATOR,
            pystray.MenuItem("Настройки...", on_settings),
            pystray.MenuItem("Выход",        on_quit),
        ),
    )
    icon.run()

def main():
    global root
    root = tk.Tk()
    root.withdraw()

    rebind_hotkeys()

    if not os.path.exists(CONFIG_FILE):
        root.after(300, open_settings)

    print("OneKey Switch запущен")
    print(f"  {config['key_ru']} → RU | {config['key_en']} → ENG | "
          f"{config['key_copy']} → COPY | {config['key_paste']} → PASTE")
    print("  Правый клик на иконке в трее → Настройки / Выход")

    threading.Thread(target=start_tray, daemon=True).start()
    root.mainloop()

if __name__ == "__main__":
    main()
