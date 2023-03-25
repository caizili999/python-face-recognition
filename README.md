# python-face-recognition
基友python开发的人脸识别签到test
# 前言

本项目为IOT实验室人员签到考勤设计，系统实现功能：
**1.人员人脸识别并完成签到/签退
2.考勤时间计算
3.保存考勤数据为CSV格式（Excel表格）**

*PS：本系统2D人脸识别，节约了繁琐的人脸识别训练部分，简洁快捷*

**该项目为测试版，正式版会加入更多的功能，持续更新中.....**
*测试版项目地址我会放到结尾*
![image](https://user-images.githubusercontent.com/89065432/226946199-18f99779-ee18-49ae-b961-d692c0f42838.png)
![image](https://user-images.githubusercontent.com/89065432/226946375-2c72123e-c7d7-43b3-acbb-a86da469d07c.png)
![image](https://user-images.githubusercontent.com/89065432/226946448-e2ae30f1-7e19-4280-99fa-2fef9b9deda3.png)
![image](https://user-images.githubusercontent.com/89065432/226946314-abdfbd77-cb0a-4a88-bd74-9564fd11f295.png)
![image](https://user-images.githubusercontent.com/89065432/226946505-af68bb86-69d4-445b-993d-415cce6ea2bf.png)
![image](https://user-images.githubusercontent.com/89065432/226946580-a040f8a9-c9d5-40ed-9087-fbbd78874d24.png)
![image](https://user-images.githubusercontent.com/89065432/226946690-2fcfd32b-d026-409f-8210-c06505d07a80.png)

# 项目需要的环境

*核心环境：*
**OpenCV-Python     			 4.5.5.64**
**face_recognition					1.30
face_recognition_model   				0.3.0
dlib 								19.23.1**

*UI窗体界面:*
**PyQt5                        5.15.4
pyqt5-plugins                5.15.4.2.2
PyQt5-Qt5                    5.15.2
PyQt5-sip                    12.10.1
pyqt5-tools                  5.15.4.3.2**
辅助开发QT-designer
![image](https://user-images.githubusercontent.com/89065432/226945877-aaa46b45-8700-4b16-bfff-f2abe10f0715.png)
![image](https://user-images.githubusercontent.com/89065432/226945836-70001d06-592d-4f3b-a66a-ae3a0c821406.png)
![image](https://user-images.githubusercontent.com/89065432/226947812-e520cbe6-5a3b-4491-8ba4-e7b5c0688459.png)

# 代码部分

## 核心代码

**MainWindow.py**
*UI文件加载：*

```python
class Ui_Dialog(QDialog):
    def __init__(self):
        super(Ui_Dialog, self).__init__()
        loadUi("mainwindow.ui", self)       #加载QTUI文件

        self.runButton.clicked.connect(self.runSlot)

        self._new_window = None
        self.Videocapture_ = None
```

*摄像头调用：*

```python
    def refreshAll(self):
        print("当前调用人俩检测摄像头编号（0为笔记本内置摄像头，1为USB外置摄像头）：")
        self.Videocapture_ = "0"
```

**OutWindow.py**
*获取当前系统时间*

```python
class Ui_OutputDialog(QDialog):
    def __init__(self):
        super(Ui_OutputDialog, self).__init__()
        loadUi("./outputwindow.ui", self)   #加载输出窗体UI

        #datetime 时间模块
        now = QDate.currentDate()
        current_date = now.toString('ddd dd MMMM yyyy')  #时间格式
        current_time = datetime.datetime.now().strftime("%I:%M %p")
        self.Date_Label.setText(current_date)
        self.Time_Label.setText(current_time)

        self.image = None
```

*签到时间计算*

```python
    def ElapseList(self,name):
        with open('Attendance.csv', "r") as csv_file:
            csv_reader = csv.reader(csv_file, delimiter=',')
            line_count = 2

            Time1 = datetime.datetime.now()
            Time2 = datetime.datetime.now()
            for row in csv_reader:
                for field in row:
                    if field in row:
                        if field == 'Clock In':
                            if row[0] == name:
                                Time1 = (datetime.datetime.strptime(row[1], '%y/%m/%d %H:%M:%S'))
                                self.TimeList1.append(Time1)
                        if field == 'Clock Out':
                            if row[0] == name:
                                Time2 = (datetime.datetime.strptime(row[1], '%y/%m/%d %H:%M:%S'))
                                self.TimeList2.append(Time2)
```

*人脸识别部分*

```python
# 人脸识别部分
        faces_cur_frame = face_recognition.face_locations(frame)
        encodes_cur_frame = face_recognition.face_encodings(frame, faces_cur_frame)

        for encodeFace, faceLoc in zip(encodes_cur_frame, faces_cur_frame):
            match = face_recognition.compare_faces(encode_list_known, encodeFace, tolerance=0.50)
            face_dis = face_recognition.face_distance(encode_list_known, encodeFace)
            name = "unknown"    #未知人脸识别为unknown
            best_match_index = np.argmin(face_dis)
            if match[best_match_index]:
                name = class_names[best_match_index].upper()
                y1, x2, y2, x1 = faceLoc
                cv2.rectangle(frame, (x1, y1), (x2, y2), (0, 255, 0), 2)
                cv2.rectangle(frame, (x1, y2 - 20), (x2, y2), (0, 255, 0), cv2.FILLED)
                cv2.putText(frame, name, (x1 + 6, y2 - 6), cv2.FONT_HERSHEY_COMPLEX, 0.5, (255, 255, 255), 1)
            mark_attendance(name)

        return frame
```

*签到数据保存与判断*

```python
# csv表格保存数据
        def mark_attendance(name):
            """
            :param name: 人脸识别部分
            :return:
            """
            if self.ClockInButton.isChecked():
                self.ClockInButton.setEnabled(False)
                with open('Attendance.csv', 'a') as f:
                        if (name != 'unknown'):         #签到判断：是否为已经识别人脸
                            buttonReply = QMessageBox.question(self, '欢迎 ' + name, '开始签到' ,
                                                               QMessageBox.Yes | QMessageBox.No, QMessageBox.No)
                            if buttonReply == QMessageBox.Yes:

                                date_time_string = datetime.datetime.now().strftime("%y/%m/%d %H:%M:%S")
                                f.writelines(f'\n{name},{date_time_string},Clock In')
                                self.ClockInButton.setChecked(False)

                                self.NameLabel.setText(name)
                                self.StatusLabel.setText('签到')
                                self.HoursLabel.setText('开始签到计时中')
                                self.MinLabel.setText('')

                                self.Time1 = datetime.datetime.now()
                                self.ClockInButton.setEnabled(True)
                            else:
                                print('签到操作失败')
                                self.ClockInButton.setEnabled(True)
            elif self.ClockOutButton.isChecked():
                self.ClockOutButton.setEnabled(False)
                with open('Attendance.csv', 'a') as f:
                        if (name != 'unknown'):
                            buttonReply = QMessageBox.question(self, '嗨呀 ' + name, '确认签退?',
                                                              QMessageBox.Yes | QMessageBox.No, QMessageBox.No)
                            if buttonReply == QMessageBox.Yes:
                                date_time_string = datetime.datetime.now().strftime("%y/%m/%d %H:%M:%S")
                                f.writelines(f'\n{name},{date_time_string},Clock Out')
                                self.ClockOutButton.setChecked(False)

                                self.NameLabel.setText(name)
                                self.StatusLabel.setText('签退')
                                self.Time2 = datetime.datetime.now()

                                self.ElapseList(name)
                                self.TimeList2.append(datetime.datetime.now())
                                CheckInTime = self.TimeList1[-1]
                                CheckOutTime = self.TimeList2[-1]
                                self.ElapseHours = (CheckOutTime - CheckInTime)
                                self.MinLabel.setText("{:.0f}".format(abs(self.ElapseHours.total_seconds() / 60)%60) + 'm')
                                self.HoursLabel.setText("{:.0f}".format(abs(self.ElapseHours.total_seconds() / 60**2)) + 'h')
                                self.ClockOutButton.setEnabled(True)
                            else:
                                print('签退操作失败')
                                self.ClockOutButton.setEnabled(True)
```

### 项目目录结构
![image](https://user-images.githubusercontent.com/89065432/226946959-f33f7ef3-a555-4b7b-a882-3ebdd5485178.png)
# 后记

***因为本系统没有进行人脸训练建立模型，系统误识别率较高，安全性较低
系统优化较差，摄像头捕捉帧数较低（8-9），后台占有高，CPU利用率较高
数据保存CSV格式，安全性较低***

