import cv2
import numpy as np
import pandas as pd
import pyzbar.pyzbar as pyzbar
import time
import datetime

# Initialize the thermal camera
thermal_camera = cv2.VideoCapture(0)

# Set the camera resolution
thermal_camera.set(cv2.CAP_PROP_FRAME_WIDTH, 160)
thermal_camera.set(cv2.CAP_PROP_FRAME_HEIGHT, 120)

# Global variables for time management
start_time_stamp = datetime.datetime.now().timestamp()
pour_start_time = None

# Initialize data storage
data = []

def get_current_ladle_mold_num():
    global start_time_stamp
        start_time = datetime.datetime.fromtimestamp(start_time_stamp)
            elapsed_time = datetime.datetime.now() - start_time
                hours = elapsed_time.seconds // 3600 + (elapsed_time.days * 24)
                    
                        if hours < 24:
                                ladle_number = int(hours // 14) + 1
                                        mold_number = hours % 14 + 1
                                            else:
                                                    start_time_stamp = datetime.datetime.now().timestamp()
                                                            ladle_number = 1
                                                                    mold_number = 1
                                                                        
                                                                            return ladle_number, mold_number

                                                                            def track_pour_time(frame, flask_number, ladle_number, mold_number, pour_temp):
                                                                                global pour_start_time, data
                                                                                    current_time = datetime.datetime.now()

                                                                                        if pour_start_time is not None:
                                                                                                pour_time = (current_time - pour_start_time).total_seconds()
                                                                                                        data.append([flask_number, ladle_number, mold_number, pour_temp, pour_time])
                                                                                                                print("Pour Time:", pour_time)
                                                                                                                    
                                                                                                                        cv2.putText(frame, f"FLASK: {flask_number}, LADLE: {ladle_number}, MOULD: {mold_number}, POUR TEMP: {pour_temp}°C", (10, 20), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 0, 255), 2)
                                                                                                                            
                                                                                                                                pour_start_time = current_time

                                                                                                                                # Main loop
                                                                                                                                num_molds = 0
                                                                                                                                flask_number = None

                                                                                                                                while True:
                                                                                                                                    ret, frame = thermal_camera.read()
                                                                                                                                        if not ret:
                                                                                                                                                break
                                                                                                                                                    
                                                                                                                                                        decoded_objects = pyzbar.decode(frame)
                                                                                                                                                            for obj in decoded_objects:
                                                                                                                                                                    flask_number = obj.data.decode()
                                                                                                                                                                            print("FLASK Number: ", flask_number)
                                                                                                                                                                                    break  # Assuming only one relevant QR code per frame
                                                                                                                                                                                        
                                                                                                                                                                                            ladle_number, mold_number = get_current_ladle_mold_num()
                                                                                                                                                                                                pour_temp = 1500  # Placeholder for actual temperature
                                                                                                                                                                                                    
                                                                                                                                                                                                        if flask_number is not None and num_molds < 14:
                                                                                                                                                                                                                track_pour_time(frame, flask_number, ladle_number, mold_number, pour_temp)
                                                                                                                                                                                                                        num_molds += 1
                                                                                                                                                                                                                            elif num_molds >= 14:
                                                                                                                                                                                                                                    num_molds = 0
                                                                                                                                                                                                                                        
                                                                                                                                                                                                                                            cv2.imshow('Thermal Video', frame)
                                                                                                                                                                                                                                                if cv2.waitKey(1) == ord('q'):
                                                                                                                                                                                                                                                        break

                                                                                                                                                                                                                                                        thermal_camera.release()
                                                                                                                                                                                                                                                        cv2.destroyAllWindows()

                                                                                                                                                                                                                                                        # Save the data
                                                                                                                                                                                                                                                        df = pd.DataFrame(data, columns=["FLASK_NUMBER", "LADLE_NUMBER", "MOLD_NUMBER", "POUR_TEMPERATURE", "POUR_TIME_SECONDS"])
                                                                                                                                                                                                                                                        df.to_excel('pour_data.xlsx', index=False)
                                                                                                                                                                                                                                                        