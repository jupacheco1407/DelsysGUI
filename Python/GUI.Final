import sys
import time
from PySide6.QtWidgets import (
    QApplication, QWidget, QVBoxLayout, QPushButton, QLabel, QProgressBar,
    QStackedWidget, QMainWindow, QMessageBox, QScrollArea, QFrame, QLineEdit, 
    QFormLayout, QComboBox
)
from PySide6.QtCore import Qt, QThread, Signal, QTimer
from PySide6.QtGui import QFont, QPalette, QColor

from AeroPy.TrignoBase import TrignoBase
from AeroPy.DataManager import DataKernel

emg_guids = [] 
max_force_by_channel = {}
sensor_mapping = {}
movement_name = ""

class ForceMonitorThread(QThread):
    force_updated = Signal(str, float)

    def __init__(self, base):
        super().__init__()
    #Monitora EMG data e calcula a porcentagem de força usada
    def run(self):
        self.running = True
        while self.running:
            if self.base.TrigBase.CheckYTDataQueue():
                yt_data = self.base.TrigBase.PollYTData()
                for channel_guid, data in yt_data.items():
                    if channel_guid in emg_guids:
                        max_val = 0
                        for value_tuple in data:
                            value = abs(value_tuple.Item2)
                            if value > max_val:
                                max_val = value
                        if channel_guid in max_force_by_channel and max_force_by_channel[channel_guid] > 0:
                            percent = (max_val / max_force_by_channel[channel_guid]) * 100
                            self.force_updated.emit(str(channel_guid), percent)
            self.msleep(200)  

    def stop(self):
        self.running = False
        self.quit()
        self.wait()

class ConfigPage(QWidget):
    configured = Signal(str, dict)

    #Configura os sensores e mapeia os músculos
    def __init__(self, guids):
        super().__init__()
        self.guids = guids
        self.mapping = {}

        layout = QVBoxLayout(self)

        self.label = QLabel("Configure Movement")
        self.label.setFont(QFont("Arial", 20, QFont.Bold))
        self.label.setAlignment(Qt.AlignCenter)
        self.label.setStyleSheet("color: white;")

        self.name_input = QLineEdit()
        self.name_input.setPlaceholderText("Movement name")
        self.name_input.setStyleSheet("""
            QLineEdit {
                background-color: #333;
                color: white;
                border: 1px solid #555;
                padding: 5px;
                border-radius: 4px;
            }
        """)

        self.form_layout = QFormLayout()
        self.muscle_inputs = {}
        self.expectation_inputs = {}
        self.reference_inputs = {}  

        for guid in guids:
            muscle_input = QLineEdit()
            muscle_input.setStyleSheet("""
                QLineEdit {
                    background-color: #333;
                    color: white;
                    border: 1px solid #555;
                    padding: 5px;
                    border-radius: 4px;
                }
            """)
            
            expectation_input = QComboBox()
            expectation_input.addItems(["Expected", "Not Expected"])
            expectation_input.setStyleSheet("""
                QComboBox {
                    background-color: #333;
                    color: white;
                    border: 1px solid #555;
                    padding: 5px;
                    border-radius: 4px;
                }
                QComboBox QAbstractItemView {
                    background-color: #333;
                    color: white;
                    selection-background-color: #555;
                }
            """)
            
            reference_input = QComboBox() 
            reference_input.addItems([f"Sensor {g}" for g in guids])
            reference_input.setEnabled(False)
            reference_input.setStyleSheet("""
                QComboBox {
                    background-color: #222;
                    color: #777;
                    border: 1px solid #444;
                    padding: 5px;
                    border-radius: 4px;
                }
            """)

            expectation_input.currentTextChanged.connect(
                lambda text, ref=reference_input: ref.setEnabled(text == "Not Expected")
            )

            self.form_layout.addRow(QLabel(f"Sensor {guid} - Muscle:"), muscle_input)
            self.form_layout.addRow(QLabel(f"Sensor {guid} - Activation:"), expectation_input)
            self.form_layout.addRow(QLabel(f"Sensor {guid} - Reference Sensor:"), reference_input)
            
            for i in range(self.form_layout.rowCount()):
                item = self.form_layout.itemAt(i, QFormLayout.LabelRole)
                if item and item.widget():
                    item.widget().setStyleSheet("color: white;")

            self.muscle_inputs[guid] = muscle_input
            self.expectation_inputs[guid] = expectation_input
            self.reference_inputs[guid] = reference_input

        self.button = QPushButton("Continue")
        self.button.setStyleSheet("""
            QPushButton {
                background-color: #4CAF50;
                color: white;
                border: none;
                padding: 10px;
                border-radius: 5px;
                font-weight: bold;
            }
            QPushButton:hover {
                background-color: #45a049;
            }
        """)
        self.button.clicked.connect(self.save_config)

        layout.addWidget(self.label)
        layout.addWidget(self.name_input)
        layout.addLayout(self.form_layout)
        layout.addWidget(self.button)

    def save_config(self):
        movement = self.name_input.text()
        mapping = {}
        for guid in self.guids:
            muscle = self.muscle_inputs[guid].text()
            expected = self.expectation_inputs[guid].currentText() == "Expected"
            reference_guid = None
            if not expected:
                ref_text = self.reference_inputs[guid].currentText()
                reference_guid = ref_text.split()[-1] 
            mapping[str(guid)] = {
                "muscle": muscle,
                "expected": expected,
                "reference": reference_guid if not expected else None
            }
        self.configured.emit(movement, mapping)

class ConnectPage(QWidget):
    connected = Signal(object, list)

    #Conecta ao Trigno Base e escaneia os sensores
    def __init__(self):
        super().__init__()
        layout = QVBoxLayout(self)
        self.label = QLabel("Connect to Trigno Base")
        self.label.setFont(QFont("Arial", 20, QFont.Bold))
        self.label.setAlignment(Qt.AlignCenter)
        self.label.setStyleSheet("color: white;")

        self.button = QPushButton("Connect")
        self.button.setStyleSheet("""
            QPushButton {
                background-color: #4CAF50;
                color: white;
                border: none;
                padding: 10px;
                border-radius: 5px;
                font-weight: bold;
            }
            QPushButton:hover {
                background-color: #45a049;
            }
        """)
        
        self.status = QLabel("")
        self.status.setAlignment(Qt.AlignCenter)
        self.status.setStyleSheet("color: white;")

        layout.addWidget(self.label)
        layout.addWidget(self.button)
        layout.addWidget(self.status)

        self.button.clicked.connect(self.connect_to_base)

    def connect_to_base(self):
        try:
            base = TrignoBase(None)
            data_handler = DataKernel(base)
            base.collection_data_handler = data_handler
            base.Connect_Callback()
            sensors = base.Scan_Callback()

            local_guids = []
            for sensor in sensors:
                for channel in sensor.TrignoChannels:
                    if str(channel.Type) == 'EMG':
                        local_guids.append(channel.Id)

            base.TrigBase.Configure(start_trigger=False, stop_trigger=False)
            self.status.setText("Connected successfully.")
            self.connected.emit(base, local_guids)
        except Exception as e:
            self.status.setText(f"Connection failed: {e}")

class CalibratePage(QWidget):
    calibrated = Signal()

    def __init__(self, base, guids):
        super().__init__()
        self.base = base
        self.guids = guids
        self.current_sensor_index = 0
        self.calibration_values = {}
        self.is_calibrating = False
        
        layout = QVBoxLayout(self)

        self.label = QLabel("Calibrate Maximum Force")
        self.label.setFont(QFont("Arial", 20, QFont.Bold))
        self.label.setAlignment(Qt.AlignCenter)
        self.label.setStyleSheet("color: white;")

        self.sensor_label = QLabel("")
        self.sensor_label.setAlignment(Qt.AlignCenter)
        self.sensor_label.setStyleSheet("color: white;")

        self.instruction = QLabel("")
        self.instruction.setAlignment(Qt.AlignCenter)
        self.instruction.setStyleSheet("color: white;")

        self.progress_bar = QProgressBar()
        self.progress_bar.setRange(0, 100)
        self.progress_bar.setAlignment(Qt.AlignCenter)
        self.progress_bar.setStyleSheet("""
            QProgressBar {
                border: 1px solid #444;
                border-radius: 5px;
                background-color: #333;
                color: white;
                text-align: center;
            }
            QProgressBar::chunk {
                background-color: #2196F3;
                border-radius: 5px;
            }
        """)

        self.button = QPushButton("Start Calibration")
        self.button.setStyleSheet("""
            QPushButton {
                background-color: #2196F3;
                color: white;
                border: none;
                padding: 10px;
                border-radius: 5px;
                font-weight: bold;
            }
            QPushButton:hover {
                background-color: #0b7dda;
            }
        """)
        
        self.result = QLabel("")
        self.result.setAlignment(Qt.AlignCenter)
        self.result.setStyleSheet("color: white;")

        layout.addWidget(self.label)
        layout.addWidget(self.sensor_label)
        layout.addWidget(self.instruction)
        layout.addWidget(self.progress_bar)
        layout.addWidget(self.button)
        layout.addWidget(self.result)

        self.button.clicked.connect(self.start_calibration)
        self.update_sensor_display()

    def update_sensor_display(self):
        if self.current_sensor_index < len(self.guids):
            current_guid = self.guids[self.current_sensor_index]
            self.sensor_label.setText(f"Calibrating Sensor: {current_guid}")
            self.instruction.setText("Apply maximum force for this sensor for 5 seconds.")
        else:
            self.sensor_label.setText("All sensors calibrated!")
            self.instruction.setText("")

    def start_calibration(self):
        if not self.is_calibrating:
            self.is_calibrating = True
            self.button.setEnabled(False)
            self.button.setText("Calibrating...")
            self.result.setText("Measuring...")
            
            self.base.TrigBase.Start(ytdata=True)
            self.start_time = time.time()
            
            self.timer = QTimer()
            self.timer.timeout.connect(self.poll_data)
            self.timer.start(50)
        else:
            self.stop_calibration()

    #Calibra a força máxima do sensor atual
    def poll_data(self):
        if time.time() - self.start_time >= 5:
            self.stop_calibration()
            return

        if self.base.TrigBase.CheckYTDataQueue():
            yt_data = self.base.TrigBase.PollYTData()
            current_guid = self.guids[self.current_sensor_index]
            
            if current_guid in yt_data:
                data = yt_data[current_guid]
                max_val = 0
                for value_tuple in data:
                    value = abs(value_tuple.Item2)
                    if value > max_val:
                        max_val = value
                
                if current_guid not in self.calibration_values or max_val > self.calibration_values.get(current_guid, 0):
                    self.calibration_values[current_guid] = max_val
                
                progress = int((time.time() - self.start_time) / 5 * 100)
                self.progress_bar.setValue(progress)

    def stop_calibration(self):
        self.timer.stop()
        self.base.TrigBase.Stop()
        
        current_guid = self.guids[self.current_sensor_index]
        if current_guid in self.calibration_values:
            global max_force_by_channel
            max_force_by_channel[current_guid] = self.calibration_values[current_guid]
            self.result.setText(f"Max force for {current_guid}: {self.calibration_values[current_guid]:.2f}")
        else:
            self.result.setText(f"No data captured for {current_guid}")
        
        self.is_calibrating = False
        self.button.setEnabled(True)
        self.button.setText("Next Sensor")
        self.progress_bar.setValue(0)
        
        #Passa para o próximo sensor ou termina a calibração
        self.current_sensor_index += 1
        if self.current_sensor_index < len(self.guids):
            self.update_sensor_display()
            self.button.setText("Start Calibration")
        else:
            self.sensor_label.setText("All sensors calibrated!")
            self.instruction.setText("Calibration complete!")
            self.button.setEnabled(False)
            self.calibrated.emit()

class MonitorPage(QWidget):
    def __init__(self, base, movement, mapping):
        super().__init__()
        self.base = base
        self.movement_name = movement
        self.mapping = mapping
        self.thread = None
        self.progress_bars = {}
        self.alert_labels = {}  
        self.latest_values = {}

        layout = QVBoxLayout(self)

        title = QLabel(f"Monitoring: {movement}")
        title.setFont(QFont("Arial", 20, QFont.Bold))
        title.setAlignment(Qt.AlignCenter)
        title.setStyleSheet("color: white;")
        layout.addWidget(title)

        scroll = QScrollArea()
        scroll.setWidgetResizable(True)
        scroll_content = QWidget()
        self.bars_layout = QVBoxLayout(scroll_content)
        scroll.setWidget(scroll_content)
        scroll.setStyleSheet("""
            QScrollArea {
                border: none;
            }
            QWidget {
                background-color: transparent;
            }
        """)

        self.global_alert_label = QLabel("") 
        self.global_alert_label.setStyleSheet("color: #FF5252; font-weight: bold;")
        self.global_alert_label.setAlignment(Qt.AlignCenter)
        layout.addWidget(scroll)
        layout.addWidget(self.global_alert_label)

        self.button = QPushButton("Start Monitoring")
        self.button.setStyleSheet("""
            QPushButton {
                background-color: #2196F3;
                color: white;
                border: none;
                padding: 10px;
                border-radius: 5px;
                font-weight: bold;
            }
            QPushButton:hover {
                background-color: #0b7dda;
            }
        """)
        self.button.clicked.connect(self.start_monitoring)
        layout.addWidget(self.button)

    #Cria as barras de progresso para cada sensor
    def create_bar(self, channel_guid):
        bar = QProgressBar()
        bar.setRange(0, 100)
        bar.setFormat(f"{self.mapping[channel_guid]['muscle']}: %p%")
        bar.setAlignment(Qt.AlignCenter)
        bar.setFixedHeight(25)
        self._set_bar_color(bar, "#4CAF50")
        self.progress_bars[str(channel_guid)] = bar
        self.bars_layout.addWidget(bar)

        alert_label = QLabel("")
        alert_label.setStyleSheet("color: #FF5252; font-weight: bold;")
        alert_label.setAlignment(Qt.AlignCenter)
        self.alert_labels[str(channel_guid)] = alert_label
        self.bars_layout.addWidget(alert_label)

    def _set_bar_color(self, bar, color):
        bar.setStyleSheet(f"""
            QProgressBar {{
                border: 1px solid #444;
                border-radius: 5px;
                background-color: #333;
                color: white;
                text-align: center;
            }}
            QProgressBar::chunk {{
                background-color: {color};
                border-radius: 5px;
            }}
        """)

    def update_display(self, guid, percent):
        if guid in self.progress_bars:
            bar = self.progress_bars[guid]
            bar.setValue(int(percent))
            
            config = self.mapping[guid] 
            if config["expected"]:
                if percent > 70:
                    color = "#1f6722" 
                elif percent > 40:
                    color = "#4caf50"  
                else:
                    color = "#a5d6a7"  
            else:
                reference_guid = config["reference"]
                reference_percent = self.latest_values.get(reference_guid, 0)
                if reference_percent == 0:
                    ratio = 0
                else:
                    ratio = (percent / reference_percent) * 100 
                if ratio < 30:
                    color = "#4caf50"
                elif ratio < 70:
                    color = "#ffeb3b" 
                else:
                    color = "#8d140b"  
            
            self._set_bar_color(bar, color)
            self.latest_values[guid] = percent
        
        self.check_for_incorrect_activation()

    #Verifica se há ativação incorreta dos músculos
    def check_for_incorrect_activation(self):
        for alert_label in self.alert_labels.values():
            alert_label.setText("")

        for guid, config in self.mapping.items():
            if not config["expected"] and config["reference"]:
                current_val = self.latest_values.get(guid, 0)
                reference_val = self.latest_values.get(config["reference"], 0)

                if current_val > reference_val and current_val > 20:
                    muscle_name = config["muscle"]
                    reference_muscle = self.mapping[config["reference"]]["muscle"]
                    alert_msg = f"⚠ {muscle_name} > {reference_muscle}"
                    self.alert_labels[guid].setText(alert_msg)
                    self.global_alert_label.setText("Incorrect activation detected!")

    def start_monitoring(self):
        if not max_force_by_channel or len(max_force_by_channel) != len(emg_guids):
            msg = QMessageBox()
            msg.setIcon(QMessageBox.Warning)
            msg.setText("Please calibrate all sensors first.")
            msg.setWindowTitle("Error")
            msg.setStyleSheet("""
                QMessageBox {
                    background-color: #222;
                }
                QLabel {
                    color: white;
                }
                QPushButton {
                    background-color: #444;
                    color: white;
                    border: none;
                    padding: 5px 10px;
                    border-radius: 4px;
                }
            """)
            msg.exec()
            return

        self.base.TrigBase.Start(ytdata=True)

        for guid in emg_guids:
            if str(guid) not in self.progress_bars:
                self.create_bar(str(guid))

        self.thread = ForceMonitorThread(self.base)
        self.thread.force_updated.connect(self.update_display)
        self.thread.start()

#Pagina principal que contém todas as outras páginas
class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("EMG Force Monitor")
        self.setFixedSize(600, 600)

        dark_palette = QPalette()
        dark_palette.setColor(QPalette.Window, QColor(53, 53, 53))
        dark_palette.setColor(QPalette.WindowText, Qt.white)
        dark_palette.setColor(QPalette.Base, QColor(25, 25, 25))
        dark_palette.setColor(QPalette.AlternateBase, QColor(53, 53, 53))
        dark_palette.setColor(QPalette.ToolTipBase, Qt.white)
        dark_palette.setColor(QPalette.ToolTipText, Qt.white)
        dark_palette.setColor(QPalette.Text, Qt.white)
        dark_palette.setColor(QPalette.Button, QColor(53, 53, 53))
        dark_palette.setColor(QPalette.ButtonText, Qt.white)
        dark_palette.setColor(QPalette.BrightText, Qt.red)
        dark_palette.setColor(QPalette.Link, QColor(42, 130, 218))
        dark_palette.setColor(QPalette.Highlight, QColor(42, 130, 218))
        dark_palette.setColor(QPalette.HighlightedText, Qt.black)
        self.setPalette(dark_palette)

        self.stack = QStackedWidget()
        self.setCentralWidget(self.stack)

        self.connect_page = ConnectPage()
        self.stack.addWidget(self.connect_page)

        self.connect_page.connected.connect(self.setup_config_page)

    def setup_config_page(self, base, guids):
        global emg_guids
        emg_guids = guids
        self.base = base
        self.config_page = ConfigPage(guids)
        self.stack.addWidget(self.config_page)
        self.stack.setCurrentWidget(self.config_page)
        self.config_page.configured.connect(self.setup_next_pages)

    def setup_next_pages(self, movement, mapping):
        global sensor_mapping, movement_name
        sensor_mapping = mapping
        movement_name = movement

        self.calibrate_page = CalibratePage(self.base, emg_guids)
        self.monitor_page = MonitorPage(self.base, movement, mapping)

        self.stack.addWidget(self.calibrate_page)
        self.stack.addWidget(self.monitor_page)

        self.calibrate_page.calibrated.connect(lambda: self.stack.setCurrentWidget(self.monitor_page))
        self.stack.setCurrentWidget(self.calibrate_page)

if __name__ == "__main__":
    app = QApplication(sys.argv)
    app.setStyle("Fusion")
    window = MainWindow()
    window.show()
    sys.exit(app.exec())