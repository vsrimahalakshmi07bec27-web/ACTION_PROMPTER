# ACTION_PROMPTER
This project is a Real-Time Task Reminder System built using Arduino, a DS3231 RTC module, 4x4 Keypad, 16x2 I2C LCD, and a buzzer. The system allows users to set multiple task reminders with specific time inputs and receive alerts when the scheduled time is reached.
#include <Keypad.h>
#include <LiquidCrystal_I2C.h>
#include <Wire.h>
#include <RTClib.h>
// Define LCD at address 0x27 (or 0x3F depending on your module)
LiquidCrystal_I2C lcd(0x27, 16, 2);
// RTC module
RTC_DS3231 rtc;
// Define buzzer pin
const int buzzerPin = 10;
 // Keypad setup
const byte ROWS = 4;
const byte COLS = 4;
char keys[ROWS][COLS] = {
 {'1','2','3','A'},
 {'4','5','6','B'},
 {'7','8','9','C'},
 {'*','0','#','D'}
};
byte rowPins[ROWS] = {9, 8, 7, 6};
byte colPins[COLS] = {5, 4, 3, 2};
 Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);
// Alarm Time variables
struct Task {
 int hour;
 int minute;
 String type;
 bool set;
};
Task tasks[5]; // Array of tasks (limit to 5 tasks for now)
// Buzzer status
bool buzzerOn = false;
void setup() {
 // Start Serial for debugging
 Serial.begin(9600);
 // Start LCD and RTC
 lcd.begin(16, 2);
 lcd.backlight();
 if (!rtc.begin()) {
 lcd.print("Couldn't find RTC");
 while (1);
 }
 if (rtc.lostPower()) {
 lcd.print("RTC lost power");
 rtc.adjust(DateTime(F(__DATE__), F(__TIME__))); // Set to compile time
 }
 // Buzzer pin output
 pinMode(buzzerPin, OUTPUT);
 digitalWrite(buzzerPin, LOW);
 lcd.setCursor(0, 0);
 lcd.print("System Ready");
 delay(2000);
 lcd.clear();
 // Initialize tasks array
 for (int i = 0; i < 5; i++) {
 tasks[i].set = false; // Mark all tasks as not set
 }
}
void loop() {
 DateTime now = rtc.now();
 // Show time and date
 lcd.setCursor(0, 0);
 lcd.print(now.hour() < 10 ? "0" : ""); lcd.print(now.hour());
 lcd.print(":");
 lcd.print(now.minute() < 10 ? "0" : ""); lcd.print(now.minute());
 lcd.print(":");
 lcd.print(now.second() < 10 ? "0" : ""); lcd.print(now.second());
 lcd.setCursor(0, 1);
 lcd.print(now.day() < 10 ? "0" : ""); lcd.print(now.day());
 lcd.print("/");
 9
 lcd.print(now.month() < 10 ? "0" : ""); lcd.print(now.month());
 lcd.print("/");
 lcd.print(now.year() % 100);
 delay(500);
 // Check for tasks
 checkTasks(now);
 // Keypad reading
 char key = keypad.getKey();
 if (key) {
 Serial.print("Key Pressed: ");
 Serial.println(key);
 if (key == 'A') {
 setTask();
 }
 if (key == '#') {
 stopAlarm();
 }
 if (key == '*') {
 showTasks();
 }
 }
}
// Function to check and trigger tasks
void checkTasks(DateTime now) {
 for (int i = 0; i < 5; i++) {
 if (tasks[i].set && tasks[i].hour == now.hour() && tasks[i].minute == now.minute() &&
!buzzerOn) {
 digitalWrite(buzzerPin, HIGH);
 buzzerOn = true;
 lcd.clear();
 lcd.setCursor(0, 0);
 lcd.print("Task: ");
 lcd.print(tasks[i].type);
 lcd.setCursor(0, 1);
 lcd.print("Press # to Stop");
 }
 }
}
// Function to set task
void setTask() {
 lcd.clear();
 lcd.setCursor(0, 0);
 lcd.print("Set Hour (00-23)");
 int hour = readNumber(2);
 lcd.clear();
 lcd.print("Set Minute (00-59)");
 int minute = readNumber(2);
 lcd.clear();
 lcd.print("1.Assign 2.Test");
 lcd.setCursor(0, 1);
 lcd.print("3.Task 4.Other");
 String taskType = "";
 while (true) {
 char k = keypad.getKey();
 if (k) {
 if (k == '1') taskType = "Assignment";
 else if (k == '2') taskType = "Test";
 else if (k == '3') taskType = "Task";
 else if (k == '4') taskType = "AnyOther";
 else taskType = "None";
 break;
 }
 }
 // Find first empty slot
 for (int i = 0; i < 5; i++) {
 if (!tasks[i].set) {
 tasks[i].hour = hour;
 tasks[i].minute = minute;
 tasks[i].type = taskType;
 tasks[i].set = true;
 break;
 }
 }
 lcd.clear();
 lcd.print("Task Set:");
 lcd.setCursor(0, 1);
 lcd.print(hour < 10 ? "0" : ""); lcd.print(hour);
 lcd.print(":");
 lcd.print(minute < 10 ? "0" : ""); lcd.print(minute);
 delay(2000);
 lcd.clear();
}
// Function to stop the alarm
void stopAlarm() {
 digitalWrite(buzzerPin, LOW);
 buzzerOn = false;
 lcd.clear();
}
// Show stored tasks
void showTasks() {
 lcd.clear();
 lcd.setCursor(0, 0);
 lcd.print("Stored Tasks:");

 for (int i = 0; i < 5; i++) {
 if (tasks[i].set) {
 lcd.setCursor(0, 1);
 lcd.print("T");
 lcd.print(i + 1);
 lcd.print(": ");
 11
 lcd.print(tasks[i].type);
 lcd.print(" ");
 lcd.print(tasks[i].hour < 10 ? "0" : ""); lcd.print(tasks[i].hour);
 lcd.print(":");
 lcd.print(tasks[i].minute < 10 ? "0" : ""); lcd.print(tasks[i].minute);
 delay(2000); // Delay to view tasks one by one
 }
 }
}
// Read multiple digits number from keypad
int readNumber(byte digits) {
 String number = "";
 while (number.length() < digits) {
 char key = keypad.getKey();
 if (key >= '0' && key <= '9') {
 number += key;
 lcd.setCursor(number.length() - 1, 1);
 lcd.print(key);
 delay(200); // Added small delay for stability
 }
 }
 return number.toInt();
}
