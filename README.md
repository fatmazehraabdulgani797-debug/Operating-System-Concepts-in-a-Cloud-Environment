# Operating-System-Concepts-in-a-Cloud-Environment
مفاهيم أنظمة التشغيل في بيئة سحابية
import sys
import random
import numpy as np
from PyQt6.QtWidgets import (QApplication, QMainWindow, QWidget, QVBoxLayout, 
                             QHBoxLayout, QLabel, QPushButton, QLineEdit, 
                             QComboBox, QTableWidget, QTableWidgetItem, QHeaderView,
                             QStackedWidget, QTextEdit, QFrame, QMessageBox, QSpinBox)
from PyQt6.QtCore import Qt, QTimer
from PyQt6.QtGui import QFont

import matplotlib
matplotlib.use('QtAgg')
from matplotlib.backends.backend_qtagg import FigureCanvasQTAgg as FigureCanvas
import matplotlib.pyplot as plt

# =========================================================================
# 0. حاوية الرسوم البيانية المدمجة لـ Matplotlib داخل PyQt6
# =========================================================================
class MtplChartCanvas(FigureCanvas):
    def __init__(self, parent=None, width=5, height=4, dpi=100, is_3d=False):
        self.fig = plt.figure(figsize=(width, height), dpi=dpi)
        if is_3d:
            self.ax = self.fig.add_subplot(111, projection='3d')
        else:
            self.ax = self.fig.add_subplot(111)
        super().__init__(self.fig)
        self.setParent(parent)
        self.apply_dark_theme(is_3d)

    def apply_dark_theme(self, is_3d=False):
        bg_color = "#1e1e2e"
        text_color = "#ffffff"
        self.fig.patch.set_facecolor(bg_color)
        self.ax.set_facecolor(bg_color)
        
        if not is_3d:
            self.ax.tick_params(colors=text_color)
            self.ax.xaxis.label.set_color(text_color)
            self.ax.yaxis.label.set_color(text_color)
            for spine in self.ax.spines.values():
                spine.set_color('#555555')
        else:
            self.ax.xaxis.set_pane_color((0.118, 0.118, 0.180, 1.0))
            self.ax.yaxis.set_pane_color((0.118, 0.118, 0.180, 1.0))
            self.ax.zaxis.set_pane_color((0.118, 0.118, 0.180, 1.0))
            self.ax.tick_params(colors=text_color)


# =========================================================================
# 1. الصفحة الرئيسية الترحيبية (Welcome Page)
# =========================================================================
class WelcomePage(QWidget):
    def __init__(self, controller):
        super().__init__()
        self.controller = controller
        layout = QVBoxLayout()
        layout.setAlignment(Qt.AlignmentFlag.AlignCenter)

        card = QFrame()
        card.setStyleSheet("background-color: #252538; border-radius: 20px; padding: 40px;")
        card_layout = QVBoxLayout(card)
        card_layout.setAlignment(Qt.AlignmentFlag.AlignCenter)

        title = QLabel("محاكي الجدولة والمعالج السيبراني الفوري\nAdvanced Cyber OS CPU Simulator")
        title.setFont(QFont("Segoe UI", 22, QFont.Weight.Bold))
        title.setStyleSheet("color: #00b4d8;")
        title.setAlignment(Qt.AlignmentFlag.AlignCenter)
        card_layout.addWidget(title)

        desc = QLabel("نظام مركزي متكامل يجمع بين جدولة تعدد النوايا الحقيقية ومؤشرات الخوادم العالمية المضيئة،\n"
                      "شاشات تتبع الإحداثيات الملاحية، السجلات السريعة، وعدادات قياس الكفاءة الدائرية.")
        desc.setFont(QFont("Segoe UI", 12))
        desc.setStyleSheet("color: #e0e0e0; margin-top: 20px; margin-bottom: 30px;")
        desc.setAlignment(Qt.AlignmentFlag.AlignCenter)
        card_layout.addWidget(desc)

        start_btn = QPushButton("دخول نظام التحكم والمراقبة العالمي ⚡")
        start_btn.setFont(QFont("Segoe UI", 13, QFont.Weight.Bold))
        start_btn.setStyleSheet("background-color: #0077b6; color: white; padding: 12px 30px; border-radius: 10px;")
        start_btn.clicked.connect(lambda: self.controller.switch_page(1))
        card_layout.addWidget(start_btn)

        layout.addWidget(card)
        self.setLayout(layout)


# =========================================================================
# 2. صفحة المحاكاة والأنظمة الحية المضيئة (Simulation Page)
# =========================================================================
class SimulationPage(QWidget):
    def __init__(self, controller):
        super().__init__()
        self.controller = controller
        
        main_layout = QHBoxLayout()
        
        # --- اللوحة اليسرى: المدخلات والتحكم ---
        left_panel = QFrame()
        left_panel.setFixedWidth(280)
        left_panel.setStyleSheet("background-color: #252538; border-radius: 12px; padding: 10px;")
        left_layout = QVBoxLayout(left_panel)
        
        panel_title = QLabel("لوحة التحكم والإدخال")
        panel_title.setFont(QFont("Segoe UI", 14, QFont.Weight.Bold))
        panel_title.setStyleSheet("color: #00b4d8;")
        left_layout.addWidget(panel_title)

        input_style = "background-color: #1e1e2e; color: white; padding: 6px; border-radius: 6px; border: 1px solid #444;"
        
        self.pid_input = QLineEdit()
        self.pid_input.setPlaceholderText("اسم العملية (P1)")
        self.pid_input.setStyleSheet(input_style)
        left_layout.addWidget(self.pid_input)

        self.arrival_input = QLineEdit()
        self.arrival_input.setPlaceholderText("وقت الوصول (Arrival Time)")
        self.arrival_input.setStyleSheet(input_style)
        left_layout.addWidget(self.arrival_input)

        self.burst_input = QLineEdit()
        self.burst_input.setPlaceholderText("وقت التنفيذ (Burst Time)")
        self.burst_input.setStyleSheet(input_style)
        left_layout.addWidget(self.burst_input)

        self.priority_input = QLineEdit()
        self.priority_input.setPlaceholderText("الأولوية (Priority)")
        self.priority_input.setStyleSheet(input_style)
        left_layout.addWidget(self.priority_input)

        add_btn = QPushButton("إضافة عملية +")
        add_btn.setStyleSheet("background-color: #0077b6; color: white; padding: 6px; border-radius: 6px; font-weight: bold;")
        add_btn.clicked.connect(self.add_process)
        left_layout.addWidget(add_btn)

        rand_btn = QPushButton("توليد عشوائي سريـع 🎲")
        rand_btn.setStyleSheet("background-color: #5c5c7d; color: white; padding: 6px; border-radius: 6px;")
        rand_btn.clicked.connect(self.generate_random_processes)
        left_layout.addWidget(rand_btn)

        sep = QFrame()
        sep.setFrameShape(QFrame.Shape.HLine)
        sep.setStyleSheet("color: #444; margin: 5px 0;")
        left_layout.addWidget(sep)

        core_lbl = QLabel("عدد النوايا (Cores):")
        core_lbl.setStyleSheet("color: white; font-weight: bold;")
        left_layout.addWidget(core_lbl)
        
        self.core_spin = QSpinBox()
        self.core_spin.setRange(1, 4)
        self.core_spin.setStyleSheet(input_style)
        left_layout.addWidget(self.core_spin)

        algo_lbl = QLabel("اختر الخوارزمية:")
        algo_lbl.setStyleSheet("color: white; font-weight: bold;")
        left_layout.addWidget(algo_lbl)

        self.algo_combo = QComboBox()
        self.algo_combo.addItems(["FCFS", "SJF (Non-preemptive)", "Priority", "Round Robin"])
        self.algo_combo.setStyleSheet("background-color: #1e1e2e; color: white; padding: 6px; border-radius: 6px;")
        self.algo_combo.currentTextChanged.connect(self.toggle_quantum)
        left_layout.addWidget(self.algo_combo)

        self.quantum_input = QLineEdit()
        self.quantum_input.setPlaceholderText("Time Quantum")
        self.quantum_input.setStyleSheet(input_style)
        self.quantum_input.setEnabled(False)
        left_layout.addWidget(self.quantum_input)

        ctrl_lbl = QLabel("المحاكاة الفورية:")
        ctrl_lbl.setStyleSheet("color: #ff7f0e; font-weight: bold; margin-top: 5px;")
        left_layout.addWidget(ctrl_lbl)

        self.run_btn = QPushButton("تشغيل تلقائي ▶️")
        self.run_btn.setStyleSheet("background-color: #2ca02c; color: white; padding: 6px; border-radius: 6px; font-weight: bold;")
        self.run_btn.clicked.connect(self.start_live_simulation)
        left_layout.addWidget(self.run_btn)

        self.pause_btn = QPushButton("إيقاف مؤقت ⏸")
        self.pause_btn.setStyleSheet("background-color: #ff7f0e; color: white; padding: 6px; border-radius: 6px;")
        self.pause_btn.clicked.connect(self.pause_live_simulation)
        left_layout.addWidget(self.pause_btn)

        clear_btn = QPushButton("مسح البيانات 🗑️")
        clear_btn.setStyleSheet("background-color: #d62728; color: white; padding: 6px; border-radius: 6px;")
        clear_btn.clicked.connect(self.clear_all_data)
        left_layout.addWidget(clear_btn)
        
        left_layout.addStretch()

        # --- اللوحة اليمنى: جدول النتائج ومخطط غانت ومراقبة السيرفرات والترميز ---
        right_panel = QWidget()
        right_layout = QVBoxLayout(right_panel)
        
        # [ميزة 1]: شريط مؤشرات الخوادم العالمية المضيئة (مربعات صغيرة ونقطة تومض وضغط متغير)
        servers_frame = QFrame()
        servers_frame.setStyleSheet("background-color: #11111b; border-radius: 10px; padding: 8px;")
        servers_layout = QHBoxLayout(servers_frame)
        servers_layout.addWidget(QLabel("🌐 الحركية الدولية للأحمال:"))
        
        self.server_widgets = {}
        regions = [("US", "أمريكا"), ("EU", "أوروبا"), ("RU", "روسيا"), ("AS", "آسيا")]
        for code, name in regions:
            lbl_box = QLabel(f"🟢 {name}: 0%")
            lbl_box.setStyleSheet("color: #00ff00; font-weight: bold; background-color: #252538; padding: 6px 12px; border-radius: 6px; border: 1px solid #333;")
            servers_layout.addWidget(lbl_box)
            self.server_widgets[code] = lbl_box
            
        right_layout.addWidget(servers_frame)
        
        # جدول البيانات المركزي للعمليات
        self.table = QTableWidget(0, 6)
        self.table.setHorizontalHeaderLabels(["ID", "Arrival", "Burst", "Priority", "Waiting", "Turnaround"])
        self.table.horizontalHeader().setSectionResizeMode(QHeaderView.ResizeMode.Stretch)
        self.table.setStyleSheet("QTableWidget { background-color: #252538; color: white; gridline-color: #444; border-radius: 10px; }"
                                 "QHeaderView::section { background-color: #1e1e2e; color: white; padding: 5px; border: 1px solid #444; }")
        right_layout.addWidget(self.table)

        # [ميزة 2]: شاشة سجل العمليات الفوري السوداء (Fast Cyber Terminal Log) نصوص خضراء متحركة بسرعة
        log_lbl = QLabel("📟 شاشة سجل العمليات الفوري السيبراني (نصوص متدفقة):")
        log_lbl.setStyleSheet("color: #00b4d8; font-weight: bold; font-size: 11px;")
        right_layout.addWidget(log_lbl)
        self.terminal_log = QTextEdit()
        self.terminal_log.setReadOnly(True)
        self.terminal_log.setFixedHeight(110)
        self.terminal_log.setStyleSheet("background-color: #000000; color: #00ff00; font-family: 'Courier New'; font-size: 11px; border: 1px solid #222; padding: 5px;")
        right_layout.addWidget(self.terminal_log)

        # الرسوم البيانية المزدوجة (مخطط غانت ومؤشر الكفاءة الدائري)
        charts_box = QHBoxLayout()
        self.gantt_canvas = MtplChartCanvas(self, width=4.5, height=2.5)
        charts_box.addWidget(self.gantt_canvas, stretch=3)
        
        # [ميزة 3]: عداد كفاءة النظام الدائري مثل عداد السيارة (يتحرك المؤشر صعوداً ونزولاً)
        self.gauge_canvas = MtplChartCanvas(self, width=2.5, height=2.5)
        charts_box.addWidget(self.gauge_canvas, stretch=2)
        
        right_layout.addLayout(charts_box)

        main_layout.addWidget(left_panel)
        main_layout.addWidget(right_panel)
        self.setLayout(main_layout)

        # التايمر المستقل للمحاكاة وتحديث المؤشرات والعداد
        self.sim_timer = QTimer()
        self.sim_timer.timeout.connect(self.simulation_step)
        self.current_sim_time = 0
        self.live_gantt_data = []

        # إعداد العداد الدائري عند وضع السكون
        self.draw_gauge_indicator(0)

    def toggle_quantum(self, text):
        self.quantum_input.setEnabled(text == "Round Robin")

    def add_process(self):
        try:
            pid = self.pid_input.text().strip() or f"P{len(self.controller.processes)+1}"
            arr = int(self.arrival_input.text().strip() or 0)
            brs = int(self.burst_input.text().strip())
            pr = int(self.priority_input.text().strip() or 0)
            
            self.controller.processes.append({"id": pid, "arrival": arr, "burst": brs, "priority": pr, "waiting": 0, "tat": 0})
            self.refresh_table()
            self.pid_input.clear(); self.arrival_input.clear(); self.burst_input.clear(); self.priority_input.clear()
            self.terminal_log.append(f"[SYS_LOAD]: Added Process {pid} successfully to central queue.")
        except ValueError:
            QMessageBox.critical(self, "خطأ في الإدخال", "يرجى كتابة أرقام صحيحة.")

    def generate_random_processes(self):
        for _ in range(5):
            self.controller.processes.append({
                "id": f"P{len(self.controller.processes)+1}",
                "arrival": random.randint(0, 8),
                "burst": random.randint(2, 9),
                "priority": random.randint(1, 5),
                "waiting": 0, "tat": 0
            })
        self.refresh_table()
        self.terminal_log.append("[SYS_LOAD]: Generated 5 randomized threads into execution block.")

    def refresh_table(self):
        self.table.setRowCount(0)
        for p in self.controller.processes:
            row = self.table.rowCount()
            self.table.insertRow(row)
            self.table.setItem(row, 0, QTableWidgetItem(str(p["id"])))
            self.table.setItem(row, 1, QTableWidgetItem(str(p["arrival"])))
            self.table.setItem(row, 2, QTableWidgetItem(str(p["burst"])))
            self.table.setItem(row, 3, QTableWidgetItem(str(p["priority"])))
            self.table.setItem(row, 4, QTableWidgetItem(str(p.get("waiting", "--"))))
            self.table.setItem(row, 5, QTableWidgetItem(str(p.get("tat", "--"))))
            for i in range(6):
                self.table.item(row, i).setTextAlignment(Qt.AlignmentFlag.AlignCenter)

    def clear_all_data(self):
        self.sim_timer.stop()
        self.controller.processes.clear()
        self.controller.gantt_data.clear()
        self.controller.comparison_results.clear()
        self.live_gantt_data.clear()
        self.current_sim_time = 0
        self.refresh_table()
        self.terminal_log.clear()
        self.gantt_canvas.ax.clear()
        self.gantt_canvas.apply_dark_theme()
        self.gantt_canvas.draw()
        self.draw_gauge_indicator(0)
        for code in self.server_widgets:
            self.server_widgets[code].setText(f"🟢 {code}: 0%")
            self.server_widgets[code].setStyleSheet("color: #00ff00; font-weight: bold; background-color: #252538; padding: 6px; border-radius: 6px;")

    def start_live_simulation(self):
        if not self.controller.processes:
            QMessageBox.warning(self, "تنبيه", "لا توجد عمليات لتشغيلها.")
            return
        self.num_cores = self.core_spin.value()
        self.algo = self.algo_combo.currentText()
        
        self.sim_processes = [dict(p) for p in self.controller.processes]
        for p in self.sim_processes:
            p["rem"] = p["burst"]
            p["start_time"] = -1
            p["end_time"] = -1
        
        self.current_sim_time = 0
        self.live_gantt_data = []
        self.active_cores = {c: None for c in range(self.num_cores)}
        
        try: self.quantum = int(self.quantum_input.text())
        except: self.quantum = 2
        
        self.core_quantum_timer = {c: 0 for c in range(self.num_cores)}
        self.terminal_log.append(f">> CRITICAL ENGINE INITIATED: Scheduling using [{self.algo}] on {self.num_cores} active cluster core(s).")
        self.sim_timer.start(350) # سرعة النبض والتدفق النصي الفوري

    def pause_live_simulation(self):
        if self.sim_timer.isActive():
            self.sim_timer.stop()
            self.pause_btn.setText("استئناف ▶️")
        else:
            self.sim_timer.start(350)
            self.pause_btn.setText("إيقاف مؤقت ⏸")

    def simulation_step(self):
        ready_queue = [p for p in self.sim_processes if p["arrival"] <= self.current_sim_time and p["rem"] > 0 and p not in self.active_cores.values()]
        
        if self.algo == "SJF (Non-preemptive)": ready_queue.sort(key=lambda x: x["rem"])
        elif self.algo == "Priority": ready_queue.sort(key=lambda x: x["priority"])
        elif self.algo == "Round Robin" or self.algo == "FCFS": ready_queue.sort(key=lambda x: x["arrival"])

        # تحديث الخوادم والمؤشر الدائري ديناميكياً مع حجم الضغط
        self.dynamic_cyber_pulse(len(ready_queue))

        for c in range(self.num_cores):
            if self.active_cores[c] is not None:
                p = self.active_cores[c]
                p["rem"] -= 1
                self.core_quantum_timer[c] += 1
                self.live_gantt_data.append((p["id"], self.current_sim_time, 1, c))
                
                # إدراج نصوص السجل بسرعة فائقة في الشاشة السوداء مع تحريك النص للأسفل تلقائياً
                self.terminal_log.append(f"[EXECUTION_T={self.current_sim_time}ms] Core_{c} allocating thread {p['id']} -> Rem: {p['rem']}ms")
                self.terminal_log.ensureCursorVisible()

                if p["rem"] == 0:
                    p["end_time"] = self.current_sim_time + 1
                    p["tat"] = p["end_time"] - p["arrival"]
                    p["waiting"] = p["tat"] - p["burst"]
                    for orig in self.controller.processes:
                        if orig["id"] == p["id"]:
                            orig["waiting"] = p["waiting"]
                            orig["tat"] = p["tat"]
                    self.terminal_log.append(f"✔ [SUCCESS]: Process {p['id']} completely deployed by Core_{c}.")
                    self.active_cores[c] = None
                elif self.algo == "Round Robin" and self.core_quantum_timer[c] >= self.quantum:
                    self.terminal_log.append(f"⚠ [TIMEOUT]: Quantum overflow for {p['id']} on Core_{c}. Pushed back to ready line.")
                    ready_queue.append(p)
                    self.active_cores[c] = None

            if self.active_cores[c] is None and ready_queue:
                next_p = ready_queue.pop(0)
                if next_p["start_time"] == -1: next_p["start_time"] = self.current_sim_time
                self.active_cores[c] = next_p
                self.core_quantum_timer[c] = 0

        self.refresh_table()
        self.draw_live_gantt()

        all_done = all(p["rem"] == 0 for p in self.sim_processes)
        if all_done:
            self.sim_timer.stop()
            self.terminal_log.append("🟢 [FINISH]: OPERATING CLUSTER TASK DISPATCH COMPLETED. STATUS IDLE.")
            self.draw_gauge_indicator(100) # كفاءة العداد تصل للقصوى عند الفراغ التام
            QMessageBox.information(self, "اكتملت المحاكاة", "تم تنفيذ جميع العمليات السيبرانية بنجاح على الخوادم المحددة!")
        
        self.current_sim_time += 1

    def dynamic_cyber_pulse(self, queue_len):
        # محاكاة حركة النقاط المضيئة وتغير الضغط اللحظي بجانبها
        regions = {"US": "أمريكا", "EU": "أوروبا", "RU": "روسيا", "AS": "آسيا"}
        for code in self.server_widgets:
            load = random.randint(35, 98) if queue_len > 0 else random.randint(1, 15)
            # النقطة المضيئة تومض بالأحمر 🔴 إذا زاد الضغط عن 75% وبالأخضر 🟢 لو كان مستقراً
            dot = "🔴" if load > 75 else "🟢"
            color_hex = "#ff3333" if load > 75 else "#00ff00"
            self.server_widgets[code].setText(f"{dot} {regions[code]}: {load}%")
            self.server_widgets[code].setStyleSheet(f"color: {color_hex}; font-weight: bold; background-color: #11111b; padding: 6px; border: 1px solid #444;")

        # حساب حركة المؤشر صعوداً ونزولاً لعداد الكفاءة الدائري بناءً على ضغط المهام
        eff_value = max(5, 100 - (queue_len * 18) + random.randint(-8, 8))
        eff_value = min(100, max(0, eff_value))
        self.draw_gauge_indicator(eff_value)

    def draw_gauge_indicator(self, val):
        # بناء وتحديث عداد دائري كامل لسرعة وكفاءة النظام بشكل لحظي ومتغير
        self.gauge_canvas.ax.clear()
        self.gauge_canvas.apply_dark_theme()
        ax = self.gauge_canvas.ax
        
        # القوس الدائري المحيط
        theta = np.linspace(0, np.pi, 120)
        ax.plot(np.cos(theta), np.sin(theta), color="#44445c", linewidth=4)
        
        # تقسيم النطاقات (أخضر كفاءة عالية، أحمر ضغط مرتفع)
        theta_high = np.linspace(np.pi * 0.5, np.pi, 50)
        ax.plot(np.cos(theta_high), np.sin(theta_high), color="#00ff00", linewidth=5)
        theta_low = np.linspace(0, np.pi * 0.3, 35)
        ax.plot(np.cos(theta_low), np.sin(theta_low), color="#ff3333", linewidth=5)

        # تحويل القراءة الحالية (0-100) لزاوية هندسية يتحرك معها المؤشر صعوداً ونزولاً
        angle = np.pi * (val / 100.0)
        ax.annotate('', xy=(np.cos(angle) * 0.85, np.sin(angle) * 0.85), xytext=(0, 0),
                    arrowprops=dict(arrowstyle="->", color="#ff7f0e", lw=4))
        
        ax.set_xlim(-1.1, 1.1)
        ax.set_ylim(-0.1, 1.1)
        ax.axis('off')
        ax.set_title(f"عداد كفاءة النظام اللحظي\n⚙️ ENGINE EFF: {val}%", color="white", fontname="Segoe UI", weight="bold", size=9)
        self.gauge_canvas.draw()

    def draw_live_gantt(self):
        self.gantt_canvas.ax.clear()
        self.gantt_canvas.apply_dark_theme()
        
        cmap = plt.colormaps.get_cmap("Set3")
        colors = {p["id"]: cmap(i % 12) for i, p in enumerate(self.controller.processes)}
        
        for task in self.live_gantt_data:
            pid, s, d, core_id = task
            self.gantt_canvas.ax.broken_barh([(s, d)], (core_id * 10 + 2, 6), facecolors=colors[pid], edgecolor="white", linewidth=0.5)
            self.gantt_canvas.ax.text(s + d / 2, core_id * 10 + 5, pid, ha="center", va="center", color="black", weight="bold", size=8)
        
        self.gantt_canvas.ax.set_ylim(0, self.num_cores * 10)
        self.gantt_canvas.ax.set_yticks([i * 10 + 5 for i in range(self.num_cores)])
        self.gantt_canvas.ax.set_yticklabels([f"Core {i}" for i in range(self.num_cores)], color="white")
        self.gantt_canvas.ax.set_title("مخطط غانت المتقدم لتوزيع مهام النوايا السحابية", color="white", fontname="Segoe UI")
        self.gantt_canvas.draw()


# =========================================================================
# 3. صفحة مقارنة الخوارزميات ومنحنيات الأداء (Comparison Page)
# =========================================================================
class ComparisonPage(QWidget):
    def __init__(self, controller):
        super().__init__()
        self.controller = controller
        
        layout = QVBoxLayout()
        top_bar = QHBoxLayout()
        title = QLabel("لوحة المقارنة التحليلية الشاملة للأنظمة")
        title.setFont(QFont("Segoe UI", 14, QFont.Weight.Bold))
        title.setStyleSheet("color: white;")
        top_bar.addWidget(title)
        
        calc_btn = QPushButton("بدء التحليل ومقارنة المنحنيات 📊")
        calc_btn.setStyleSheet("background-color: #ff7f0e; color: white; padding: 8px 16px; font-weight: bold; border-radius: 6px;")
        calc_btn.clicked.connect(self.compare_all_algorithms)
        top_bar.addWidget(calc_btn)
        
        layout.addLayout(top_bar)
        
        self.compare_canvas = MtplChartCanvas(self, width=8, height=4)
        layout.addWidget(self.compare_canvas)
        
        self.setLayout(layout)

    def refresh_data(self):
        self.draw_comparison()

    def compare_all_algorithms(self):
        if not self.controller.processes:
            QMessageBox.warning(self, "تنبيه", "برجاء تعبئة عمليات أولاً داخل صفحة المحاكاة.")
            return

        backup = [dict(p) for p in self.controller.processes]
        algos = ["FCFS", "SJF (Non-preemptive)", "Priority", "Round Robin"]
        self.controller.comparison_results = {}

        for algo in algos:
            procs = [dict(b) for b in backup]
            procs.sort(key=lambda x: x["arrival"])
            t, waiting_times, tat_times = 0, [], []
            
            if algo == "FCFS":
                for p in procs:
                    if t < p["arrival"]: t = p["arrival"]
                    waiting_times.append(t - p["arrival"])
                    t += p["burst"]
                    tat_times.append(t - p["arrival"])
            elif algo == "SJF (Non-preemptive)":
                unv = list(procs)
                while unv:
                    ready = [p for p in unv if p["arrival"] <= t]
                    if not ready: t = unv[0]["arrival"]; continue
                    ready.sort(key=lambda x: x["burst"])
                    p = ready[0]
                    waiting_times.append(t - p["arrival"])
                    t += p["burst"]
                    tat_times.append(t - p["arrival"])
                    unv.remove(p)
            elif algo == "Priority":
                unv = list(procs)
                while unv:
                    ready = [p for p in unv if p["arrival"] <= t]
                    if not ready: t = unv[0]["arrival"]; continue
                    ready.sort(key=lambda x: x["priority"])
                    p = ready[0]
                    waiting_times.append(t - p["arrival"])
                    t += p["burst"]
                    tat_times.append(t - p["arrival"])
                    unv.remove(p)
            elif algo == "Round Robin":
                q = 2
                rem = {p["id"]: p["burst"] for p in procs}
                arrivals = {p["id"]: p["arrival"] for p in procs}
                bursts = {p["id"]: p["burst"] for p in procs}
                t = 0
                queue = [p for p in procs if p["arrival"] == 0]
                while any(r > 0 for r in rem.values()):
                    if not queue:
                        t = min(arrivals[k] for k, v in rem.items() if v > 0)
                        queue = [p for p in procs if arrivals[p["id"]] <= t and rem[p["id"]] > 0]
                    p = queue.pop(0)
                    if rem[p["id"]] > q:
                        t += q; rem[p["id"]] -= q
                    else:
                        t += rem[p["id"]]; rem[p["id"]] = 0
                        tat = t - arrivals[p["id"]]
                        tat_times.append(tat)
                        waiting_times.append(tat - bursts[p["id"]])
                    for p_next in procs:
                        if arrivals[p_next["id"]] <= t and rem[p_next["id"]] > 0 and p_next not in queue and p_next["id"] != p["id"]:
                            queue.append(p_next)

            avg_w = sum(waiting_times) / len(waiting_times) if waiting_times else 0
            avg_t = sum(tat_times) / len(tat_times) if tat_times else 0
            self.controller.comparison_results[algo] = {"waiting": avg_w, "tat": avg_t}

        self.draw_comparison()

    def draw_comparison(self):
        self.compare_canvas.ax.clear()
        self.compare_canvas.apply_dark_theme()
        
        if not self.controller.comparison_results:
            self.compare_canvas.ax.text(0.5, 0.5, "اضغط على زر بدء التحليل لعرض منحنيات الكفاءة الهندسية", 
                                        ha="center", va="center", color="gray", size=12)
            self.compare_canvas.draw()
            return

        algos = list(self.controller.comparison_results.keys())
        avg_w = [self.controller.comparison_results[a]["waiting"] for a in algos]
        avg_t = [self.controller.comparison_results[a]["tat"] for a in algos]

        self.compare_canvas.ax.plot(algos, avg_w, color="#1f77b4", marker='o', linestyle='-', linewidth=2, label="متوسط وقت الانتظار")
        self.compare_canvas.ax.plot(algos, avg_t, color="#2ca02c", marker='s', linestyle='--', linewidth=2, label="متوسط وقت الإنجاز")
        self.compare_canvas.ax.fill_between(algos, avg_w, color="#1f77b4", alpha=0.1)
        self.compare_canvas.ax.legend()
        self.compare_canvas.ax.set_title("منحنيات مقارنة الأداء الهندسي للخوارزميات المتعددة", color="white", fontname="Segoe UI")
        self.compare_canvas.fig.tight_layout()
        self.compare_canvas.draw()


# =========================================================================
# 4. صفحة التقارير والتحليل النهائي (Report Page)
# =========================================================================
class ReportPage(QWidget):
    def __init__(self, controller):
        super().__init__()
        self.controller = controller
        
        layout = QVBoxLayout()
        title = QLabel("التقرير النهائي لكفاءة جدولة النظام والأحمال")
        title.setFont(QFont("Segoe UI", 14, QFont.Weight.Bold))
        title.setStyleSheet("color: white; margin-bottom: 10px;")
        layout.addWidget(title)

        self.textbox = QTextEdit()
        self.textbox.setReadOnly(True)
        self.textbox.setStyleSheet("background-color: #252538; color: #a9b7c6; font-family: 'Courier New'; font-size: 13px; padding: 15px; border-radius: 10px;")
        layout.addWidget(self.textbox)
        self.setLayout(layout)

    def refresh_data(self):
        self.textbox.clear()
        if not self.controller.comparison_results:
            self.textbox.setText("لم يتم إنشاء تقارير بعد. يرجى الذهاب أولاً إلى لوحة مقارنة الخوارزميات والضغط على تشغيل التحليل.")
            return

        report = "=======================================================\n"
        report += "      CORE OPERATING SYSTEM BENCHMARK REPORT           \n"
        report += "=======================================================\n\n"
        best_algo, min_wait = "", float("inf")

        for algo, stats in self.controller.comparison_results.items():
            report += f"▶️ [{algo} Scheduler]:\n"
            report += f"   - Average Waiting Time   : {stats['waiting']:.2f} ms\n"
            report += f"   - Average Turnaround Time: {stats['tat']:.2f} ms\n\n"
            if stats["waiting"] < min_wait:
                min_wait = stats["waiting"]
                best_algo = algo

        report += "-------------------------------------------------------\n"
        report += f"★ التوصية الهندسية للنظام الذكي: نوصي بـ [{best_algo}]\n"
        report += "=======================================================\n"
        self.textbox.setText(report)


# =========================================================================
# 5. صفحة الكرة الأرضية ثلاثية الأبعاد ولوحة الإحداثيات الملاحية الحية (3D Globe Page)
# =========================================================================
class Globe3DPage(QWidget):
    def __init__(self, controller):
        super().__init__()
        self.controller = controller
        
        layout = QHBoxLayout()
        
        # اللوحة اليسرى: مجسم الكرة الأرضية 3D
        left_vbox = QVBoxLayout()
        title = QLabel("مجسم محاكاة الكرة الأرضية ثلاثي الأبعاد المتزامن (3D Satellite Globe)")
        title.setFont(QFont("Segoe UI", 14, QFont.Weight.Bold))
        title.setStyleSheet("color: #00b4d8;")
        left_vbox.addWidget(title)
        
        self.globe_canvas = MtplChartCanvas(self, width=6, height=5, is_3d=True)
        left_vbox.addWidget(self.globe_canvas)
        layout.addLayout(left_vbox, stretch=3)
        
        # [ميزة 4]: اللوحة اليمنى لعرض الإحداثيات الملاحية المتغيرة باستمرار لخطوط الطول والعرض
        self.coord_panel = QFrame()
        self.coord_panel.setStyleSheet("background-color: #11111b; border-radius: 12px; border: 1px solid #333; padding: 20px;")
        coord_layout = QVBoxLayout(self.coord_panel)
        coord_layout.setAlignment(Qt.AlignmentFlag.AlignTop)
        
        coord_title = QLabel("📡 إحداثيات ومواقع تتبع البيانات:")
        coord_title.setFont(QFont("Courier New", 13, QFont.Weight.Bold))
        coord_title.setStyleSheet("color: #ff7f0e;")
        coord_layout.addWidget(coord_title)
        
        self.lat_label = QLabel("LATITUDE : 0.0000° N")
        self.lat_label.setFont(QFont("Courier New", 15, QFont.Weight.Bold))
        self.lat_label.setStyleSheet("color: #00ff00; margin-top: 25px;")
        coord_layout.addWidget(self.lat_label)
        
        self.lng_label = QLabel("LONGITUDE: 0.0000° E")
        self.lng_label.setFont(QFont("Courier New", 15, QFont.Weight.Bold))
        self.lng_label.setStyleSheet("color: #00ff00; margin-top: 15px;")
        coord_layout.addWidget(self.lng_label)
        
        self.routing_label = QLabel("CLUSTER  : STANDBY")
        self.routing_label.setFont(QFont("Courier New", 12))
        self.routing_label.setStyleSheet("color: #a9b7c6; margin-top: 35px;")
        coord_layout.addWidget(self.routing_label)
        
        layout.addWidget(self.coord_panel, stretch=1)
        self.setLayout(layout)
        
        # تايمر لتحديث زاوية رؤية الكرة وتوليد الإحداثيات المتحركة بسرعة فائقة باستمرار
        self.rotation_timer = QTimer()
        self.rotation_timer.timeout.connect(self.update_globe_and_gps_stream)
        self.angle = 0
        
    def start_globe(self):
        self.rotation_timer.start(70) # دوران سريع وتغير مستمر للإحداثيات

    def stop_globe(self):
        self.rotation_timer.stop()

    def draw_initial_globe(self):
        self.globe_canvas.ax.clear()
        self.globe_canvas.apply_dark_theme(is_3d=True)
        u = np.linspace(0, 2 * np.pi, 30)
        v = np.linspace(0, np.pi, 30)
        x = 10 * np.outer(np.cos(u), np.sin(v))
        y = 10 * np.outer(np.sin(u), np.sin(v))
        z = 10 * np.outer(np.ones(np.size(u)), np.cos(v))
        self.globe_canvas.ax.plot_surface(x, y, z, color='#0077b6', edgecolor='#00b4d8', alpha=0.35, linewidth=0.5)
        self.globe_canvas.ax.axis('off')
        self.globe_canvas.draw()

    def update_globe_and_gps_stream(self):
        # 1. تدوير محاكاة الكرة الأرضية ثلاثية الأبعاد
        self.angle = (self.angle + 2) % 360
        self.globe_canvas.ax.view_init(elev=22, azim=self.angle)
        self.globe_canvas.draw()
        
        # 2. توليد إحداثيات جغرافية سريعة الحركة تتغير باستمرار لتوضيح مكان معالجة البيانات الجارية
        rand_lat = random.uniform(-90.0, 90.0)
        rand_lng = random.uniform(-180.0, 180.0)
        lat_dir = "N" if rand_lat >= 0 else "S"
        lng_dir = "E" if rand_lng >= 0 else "W"
        
        self.lat_label.setText(f"LATITUDE : {abs(rand_lat):.4f}° {lat_dir}")
        self.lng_label.setText(f"LONGITUDE: {abs(rand_lng):.4f}° {lng_dir}")
        
        clusters = ["TOKYO_CORE_NODE", "EUROPE_MAIN_FRAME", "US_WEST_CLUSTER", "MOSCOW_DATA_CENTER", "FRANKFURT_HUB"]
        self.routing_label.setText(f"ROUTING  : {random.choice(clusters)}")


# =========================================================================
# 6. الهيكل الرئيسي وشريط التنقل الجانبي الاحترافي (Main Window)
# =========================================================================
class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("CPU Scheduling OS Cyber Simulator - Multi-Core Edition")
        self.setGeometry(80, 80, 1420, 880)
        self.setStyleSheet("background-color: #1e1e2e;")

        self.processes = []
        self.gantt_data = []
        self.comparison_results = {}

        main_widget = QWidget()
        self.setCentralWidget(main_widget)
        main_layout = QHBoxLayout(main_widget)
        main_layout.setContentsMargins(0, 0, 0, 0)
        main_layout.setSpacing(0)

        # --- شريط التنقل الجانبي (Sidebar) ---
        sidebar = QFrame()
        sidebar.setFixedWidth(240)
        sidebar.setStyleSheet("background-color: #11111b; border-right: 1px solid #333;")
        sidebar_layout = QVBoxLayout(sidebar)
        sidebar_layout.setContentsMargins(10, 30, 10, 20)

        logo = QLabel("CYBER REVOLUTION\nOS SIMULATOR")
        logo.setFont(QFont("Segoe UI", 14, QFont.Weight.Bold))
        logo.setStyleSheet("color: #00b4d8; margin-bottom: 30px;")
        logo.setAlignment(Qt.AlignmentFlag.AlignCenter)
        sidebar_layout.addWidget(logo)

        btn_style = "QPushButton { text-align: left; color: white; padding: 12px; border-radius: 8px; border: none; font-size: 13px; }" \
                    "QPushButton:hover { background-color: #252538; }"
        
        self.btn_home = QPushButton(" 🏠  الرئيسية")
        self.btn_sim = QPushButton(" 🚀  شاشة التحكم الحية والأنظمة")
        self.btn_comp = QPushButton(" 📊  مقارنة والمنحنيات")
        self.btn_rep = QPushButton(" 📋  التقارير الفنية")
        self.btn_globe = QPushButton(" 🌐  الكرة الأرضية والإحداثيات")

        for btn in [self.btn_home, self.btn_sim, self.btn_comp, self.btn_rep, self.btn_globe]:
            btn.setStyleSheet(btn_style)
            sidebar_layout.addWidget(btn)

        self.btn_home.clicked.connect(lambda: self.switch_page(0))
        self.btn_sim.clicked.connect(lambda: self.switch_page(1))
        self.btn_comp.clicked.connect(lambda: self.switch_page(2))
        self.btn_rep.clicked.connect(lambda: self.switch_page(3))
        self.btn_globe.clicked.connect(lambda: self.switch_page(4))

        sidebar_layout.addStretch()
        main_layout.addWidget(sidebar)

        # --- صفحات الواجهات المتعددة (Stacked Widget) ---
        self.pages_container = QStackedWidget()
        
        self.welcome_page = WelcomePage(self)
        self.simulation_page = SimulationPage(self)
        self.comparison_page = ComparisonPage(self)
        self.report_page = ReportPage(self)
        self.globe_page = Globe3DPage(self)

        self.pages_container.addWidget(self.welcome_page)
        self.pages_container.addWidget(self.simulation_page)
        self.pages_container.addWidget(self.comparison_page)
        self.pages_container.addWidget(self.report_page)
        self.pages_container.addWidget(self.globe_page)

        main_layout.addWidget(self.pages_container)
        self.switch_page(0)

    def switch_page(self, index):
        self.pages_container.setCurrentIndex(index)
        
        buttons = [self.btn_home, self.btn_sim, self.btn_comp, self.btn_rep, self.btn_globe]
        for i, btn in enumerate(buttons):
            if i == index:
                btn.setStyleSheet("text-align: left; color: white; padding: 12px; border-radius: 8px; background-color: #0077b6; font-weight: bold;")
            else:
                btn.setStyleSheet("text-align: left; color: white; padding: 12px; border-radius: 8px; border: none; font-size: 13px; QPushButton:hover { background-color: #252538; }")

        # تشغيل إحداثيات الملاحة ودوران الكرة الأرضية فقط عندما تكون صفحتها نشطة لحفظ أداء المعالج
        if index == 4:
            self.globe_page.draw_initial_globe()
            self.globe_page.start_globe()
        else:
            self.globe_page.stop_globe()
        if index == 2: self.comparison_page.refresh_data()
        elif index == 3: self.report_page.refresh_data()


if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = MainWindow()
    window.show()
    sys.exit(app.exec())