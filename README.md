from PyQt5.QtWidgets import *
from PyQt5.QtGui import *
from PyQt5 import uic
from PyQt5.QtCore import *

from PyQt5.QtCore import Qt,QUrl
from PyQt5.QtMultimedia import QMediaPlayer, QMediaContent
from PyQt5.QtMultimediaWidgets import QVideoWidget
from PyQt5.QtWidgets import QApplication, QMainWindow, QPushButton, QVBoxLayout, QWidget

import imutils
import cv2
import sys
import datetime


from_class = uic.loadUiType("opencv.ui")[0]

class WindowClass(QMainWindow, from_class):
    def __init__(self):
        super().__init__()
        self.setupUi(self)

        self.isCameraOn = False
        self.isRecStart = False
        self.btnRecord.hide()
        self.btnCapture.hide()

        self.pixmap = QPixmap()

        self.camera = Camera(self)
        self.camera.daemon = True

        self.record = Camera(self)
        self.record.daemon = True

        self.btnOpen.clicked.connect(self.openFile)
        self.btnCamera.clicked.connect(self.clickCamera)
        self.camera.update.connect(self.updateCamera)
        self.btnRecord.clicked.connect(self.clickRecord)
        self.btnCapture.clicked.connect(self.capture)
        self.record.update.connect(self.updateRecording)
        # self.btnSaved.clicked.connect(self.clickSave)

        self.count = 0

    def capture(self):
        self.now = datetime.datetime.now().strftime('%Y%m%d_%H%M%S')
        filename = self.now + '.png'

        # cv2.imwrite(filename, self.image)

    def updateRecording(self):
        # self.label2.setText(str(self.count))
        # self.writer.write(self.image)
        # ㄴ writer라는 객체에 write라는 함수를 
        self.count += 1
    

    def clickRecord(self):
        if self.isRecStart == False:
            self.btnRecord.setText('Rec Stop')
            self.isRecStart = True

            self.recordingStart()
        else:
            self.btnRecord.setText('Rec Start')
            self.isRecStart = False
            # self.btnRecord.hide()

            self.recordingStop()

    def recordingStart(self):
        self.record.running = True
        self.record.start()

        self.now = datetime.datetime.now().strftime('%Y%m%d_%H%M%S')
        filename = self.now + '.avi'
        self.fourcc = cv2.VideoWriter_fourcc(*'XVID')
        
        w = int(self.video.get(cv2.CAP_PROP_FRAME_WIDTH))
        h = int(self.video.get(cv2.CAP_PROP_FRAME_HEIGHT))

        self.writer = cv2.VideoWriter(filename, self.fourcc, 20.0, (w, h))

    def recordingStop(self):
        self.record.running = False    

        if self.isRecStart == True:
            self.writer.release()

    def openFile(self):
        file = QFileDialog.getOpenFileName(filter = "")

        image = cv2.imread(file[0])
        # image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)

        # h,w,c = image.shape
        # qimage = QImage(image.data, w, h, w*c, QImage.Format_RGB888)

        # self.pixmap = self.pixmap.fromImage(qimage)
        self.pixmap = self.pixmap.scaled(self.label.width(), self.label.height())
        self.label.setPixmap(self.pixmap)

    def updateCamera(self):
        retval, image = self.video.read()
        if retval:
            #
            image = cv2.cvtColor(image, cv2.COLOR_RGB2BGR)

            h,w,c = image.shape
            qimage = QImage(image.data, w, h, w * c, QImage.Format_RGB888)

            self.pixmap = self.pixmap.fromImage(qimage)
            self.pixmap = self.pixmap.scaled(self.label.width(), self.label.height())

            self.label.setPixmap(self.pixmap)

        self.count += 1

    def clickCamera(self):
        if self.isCameraOn == False:
            self.btnCamera.setText('Camera off')
            self.isCameraOn = True
            self.btnRecord.show()
            self.btnCapture.show()

            self.cameraStart()
        else:                                                                                               
            self.btnCamera.setText('Camera on')
            self.isCameraOn = False
            self.btnRecord.hide()
            self.btnCapture.hide()

            self.cameraStop()
            self.recordingStop()
    
    def cameraStart(self):
        self.camera.running = True
        self.camera.start()
        self.video = cv2.VideoCapture(-1)

    def cameraStop(self):
        self.camera.running = False
        self.count = 0
        self.video.release

    # def playVideo(self):
    #     video_path = "20231019_143240.avi"  # Replace with your video file path
    #     video_url = QMediaContent(Qt.QUrl.fromLocalFile(video_path))
    #     self.player.setMedia(video_url)
        
    #     self.player.setVideoOutput(self.video_widget)

    #     self.player.play()    

import time 

class Camera(QThread):
    update = pyqtSignal()

    def __init__(self, sec = 0, parent = None):
        super().__init__()
        self.main = parent
        self.running = True

    def run(self):
        while self.running == True:
            self.update.emit() # emit 함수는 시그널을 발생시키는 역할을 하는 함수이다
            time.sleep(0.1)    

    def stop(self):
        self.running = False

class BasicFunction():
    def __init__(self, main, gallery, capture, mode, display, pixmap, fillter):
        self.mainClass = main
        self.btnGallery = gallery
        self.btnCapture = capture
        self.btnMode = mode
        self.labelDisplay = display
        self.pixmap = pixmap
        self.btnFillter = fillter


    def openGallery(self):
        gallery = QFileDialog.getOpenFileName(self.mainClass, 'Gallery', './Capture/', '*.*')

        if gallery[0]:
            if '.png' in gallery[0]:
                image_path = gallery[0]

                image = cv2.imread(gallery[0])
                image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)

                h, w, c = image.shape
                setImage = QImage(image.data, w, h, w*c, QImage.Format_RGB888)

                self.pixmap = self.pixmap.fromImage(setImage)
                self.pixmap = self.pixmap.scaled(self.labelDisplay.size(), Qt.KeepAspectRatio)

                self.labelDisplay.setPixmap(self.pixmap)
            
            elif '.avi' in gallery[0]:
                video_path = gallery[0]
                self.cap = cv2.VideoCapture(video_path)

                frame_rate = 25

                if not self.cap.isOpened():
                    print("동영상을 열 수 없습니다.")
                    return

                while True:
                    ret, frame = self.cap.read()
                    if not ret:
                        break

                    image = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
                    h, w, c = image.shape
                    setImage = QImage(image.data, w, h, w*c, QImage.Format_RGB888)
                    self.labelDisplay.setPixmap(QPixmap.fromImage(setImage))
                    QApplication.processEvents()  # UI 업데이트

                    time.sleep(1.0 / frame_rate)

                    if cv2.waitKey() == ord('q'):
                        break

                self.cap.release() 


        else:
            print("이미지를 불러오지 못했습니다.")


    def changeMode(self):
        if self.btnCapture.text() == 'Capture':
            self.btnCapture.setText("REC Start")
        
        else:
            self.btnCapture.setText("Capture")



if __name__ == "__main__":
    app = QApplication(sys.argv)

    myWindows = WindowClass()

    myWindows.show()

    sys.exit(app.exec_())



