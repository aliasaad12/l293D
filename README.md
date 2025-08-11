# l293D
مشروع التحكم في 4 محركات DC مع سيرفو وحساس Ultrasonic باستخدام L293D

المكونات المطلوبة
	•	أردوينو Uno أو أي لوحة متوافقة
	•	شريحة L293D Motor Driver
	•	4 محركات DC
	•	محرك Servo واحد
	•	حساس Ultrasonic (HC-SR04)
	•	أسلاك توصيل + مصدر طاقة مناسب
	•	لوحة تجارب (Breadboard)

⸻

الفكرة
	1.	المرحلة الأولى:
	•	تحريك 4 محركات DC للأمام لمدة 30 ثانية.
	•	ثم للخلف لمدة دقيقة.
	•	ثم لليمين ولليسار بالتناوب لمدة دقيقة.
	2.	المرحلة الثانية:
	•	ربط محرك Servo مع الحساس Ultrasonic.
	•	إذا تم استشعار عائق على بعد 10 سم أو أقل:
	•	تتوقف المحركات.
	•	يتم تغيير اتجاه الحركة.

⸻

توصيل L293D مع 4 محركات DC
	•	المحرك الأمامي الأيسر: OUT1 و OUT2 من L293D.
	•	المحرك الأمامي الأيمن: OUT3 و OUT4.
	•	المحرك الخلفي الأيسر: شريحة L293D أخرى أو نفس الشريحة إذا كانت تدعم.
	•	المحرك الخلفي الأيمن: نفس الفكرة.
	•	أطراف التحكم:
	•	IN1, IN2 للمحرك الأول.
	•	IN3, IN4 للمحرك الثاني.
	•	وهكذا لبقية المحركات.

⸻

توصيل السيرفو
	•	الإشارة من السيرفو إلى أي مخرج PWM في الأردوينو (مثلاً D9).
	•	الطاقة: 5V من الأردوينو أو مصدر خارجي مشترك الأرضي.

⸻

توصيل الحساس Ultrasonic
	•	VCC إلى 5V.
	•	GND إلى GND.
	•	Trig إلى D7 مثلاً.
	•	Echo إلى D6.
 الكود
 #include <AFMotor.h>  // Adafruit Motor Shield library (L293D)
#include <Servo.h>
#include <stdlib.h>   // For random()

// Ultrasonic sensor pins
#define TRIG_PIN 10
#define ECHO_PIN 2

// Servo
#define SERVO_PIN 9
Servo myservo;
int pos = 90;  // Center position

// Motors (4-wheel drive)
AF_DCMotor motorFL(1);  // Front Left  (M1)
AF_DCMotor motorFR(2);  // Front Right (M2)
AF_DCMotor motorRL(3);  // Rear Left   (M3)
AF_DCMotor motorRR(4);  // Rear Right  (M4)

// Speed settings (adjust as needed)
const int forwardSpeed = 150;
const int reverseSpeed = 150;

void setup() {
  Serial.begin(9600);

  // Ultrasonic setup
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);

  // Servo setup
  myservo.attach(SERVO_PIN);
  myservo.write(pos);

  // Set motor speeds
  motorFL.setSpeed(forwardSpeed);
  motorFR.setSpeed(forwardSpeed);
  motorRL.setSpeed(forwardSpeed);
  motorRR.setSpeed(forwardSpeed);

  stopMotors();

  // Seed random
  randomSeed(analogRead(0));
}

void loop() {
  if (sweepAndCheckObstacle()) {
    avoidObstacle();
  } else {
    moveForward();
  }
}

// Sweep servo and detect obstacle
bool sweepAndCheckObstacle() {
  for (pos = 30; pos <= 110; pos += 10) {
    myservo.write(pos);
    delay(57);

    float distance = getDistance();
    Serial.print("Angle: ");
    Serial.print(pos);
    Serial.print(" | Distance: ");
    Serial.print(distance);
    Serial.println(" cm");

    if (distance > 0 && distance < 10) {
      return true;
    }
  }

  for (pos = 110; pos >= 30; pos -= 10) {
    myservo.write(pos);
    delay(57);

    float distance = getDistance();
    Serial.print("Angle: ");
    Serial.print(pos);
    Serial.print(" | Distance: ");
    Serial.print(distance);
    Serial.println(" cm");

    if (distance > 0 && distance < 10) {
      return true;
    }
  }

  return false;
}

// Avoid by reversing and turning randomly
void avoidObstacle() {
  stopMotors();
  moveBackward();
  delay(1000);
  stopMotors();
  delay(300);

  int turnDirection = random(0, 2); // 0 = left, 1 = right

  if (turnDirection == 0) {
    Serial.println("Turning LEFT to avoid obstacle");
    turnLeft();
  } else {
    Serial.println("Turning RIGHT to avoid obstacle");
    turnRight();
  }

  delay(1200);  // turn longer for clear move
  stopMotors();
  delay(300);
}

// Distance in cm using ultrasonic
float getDistance() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  long duration = pulseIn(ECHO_PIN, HIGH, 30000);
  if (duration == 0) return -1;
  return duration * 0.034 / 2;
}

// Movement commands
void moveForward() {
  motorFL.setSpeed(forwardSpeed);
  motorFR.setSpeed(forwardSpeed);
  motorRL.setSpeed(forwardSpeed);
  motorRR.setSpeed(forwardSpeed);

  motorFL.run(FORWARD);
  motorFR.run(FORWARD);
  motorRL.run(FORWARD);
  motorRR.run(FORWARD);
}

void moveBackward() {
  motorFL.setSpeed(reverseSpeed);
  motorFR.setSpeed(reverseSpeed);
  motorRL.setSpeed(reverseSpeed);
  motorRR.setSpeed(reverseSpeed);

  motorFL.run(BACKWARD);
  motorFR.run(BACKWARD);
  motorRL.run(BACKWARD);
  motorRR.run(BACKWARD);
}

void turnLeft() {
  motorFL.setSpeed(forwardSpeed);
  motorFR.setSpeed(forwardSpeed);
  motorRL.setSpeed(forwardSpeed);
  motorRR.setSpeed(forwardSpeed);

  motorFL.run(BACKWARD);
  motorFR.run(FORWARD);
  motorRL.run(BACKWARD);
  motorRR.run(FORWARD);
}

void turnRight() {
  motorFL.setSpeed(forwardSpeed);
  motorFR.setSpeed(forwardSpeed);
  motorRL.setSpeed(forwardSpeed);
  motorRR.setSpeed(forwardSpeed);

  motorFL.run(FORWARD);
  motorFR.run(BACKWARD);
  motorRL.run(FORWARD);
  motorRR.run(BACKWARD);
}

void stopMotors() {
  motorFL.run(RELEASE);
  motorFR.run(RELEASE);
  motorRL.run(RELEASE);
  motorRR.run(RELEASE);
}
