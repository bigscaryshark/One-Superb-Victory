# One-Superb-Victory
a short description


#include "Enes100.h"
#include "Tank.h"

/* sketch returns a syntax error when compiling, but runs in the simulator.
 *  Expect this is a problem with the abs<> and power<> functions we had to write, since 
 *  we can't import the math library into the simulator.
 */

double PI = 3.1416;
double TOLERANCE = PI/40; // determines heading tolerance when turning
int DELAY = 100; // determines how long OSV runs before updating location

void setup() {
  Enes100.begin("Effie", FIRE, 3, 8, 9);
  Tank.begin();
  
}

void loop() {

  // demo reel
  go(0, 0.5);
  Enes100.println("1 done");
  go(-3*PI/12, 1);
  Enes100.println("2 done");
  go(-1*PI/12, 1.5);
  Enes100.println("3 done");
  go(2*PI/12, 1.8);
  Enes100.println("4 done");
  go(0*PI/12, 2.2);
  Enes100.println("5 done");
  go(3*PI/12, 2.5);
  Enes100.println("6 done");
  go(2*PI/12, 3.1);
  Enes100.println("7 done");
  go(1*PI/12, 3.5);
  Enes100.println("8 done");
  go(-7*PI/12, 3.8);
  
  Enes100.println("I made it!"); 

}


// passed a heading and an x coordinate, function tells the OSV to move towards the 
// destination 
void go(double theta, int x){
  Enes100.updateLocation();
  while(Enes100.location.x < x){
    correct(theta);
    Enes100.updateLocation();
  }
}



// passed a heading coordinate, moves OSV in an arc 
// by  moving the wheels at different power levels. 
// Can be used to change OSV heading, but use face<> for gross corrections.
void correct(double course){
  Enes100.updateLocation();
  
  double heading = Enes100.location.theta;
    heading = Enes100.location.theta;

    // finds the difference between the intended and current heading. If number is positive, OSV
    // is facing too far left. If negative, OSV is facing too far right.
    double diff = heading - course;

    // modifies the falloff rate in the differential power equation 
    // ie, changes the severity of the curve
    int strength = (int)(  (-255.0 / power((PI/4),2) )* power(absval(diff),2) + 255 )  ;
  
    if (strength < -255) strength = -255; 
    // caps minimum power that can be sent to a motor at -255)

      /*
      Enes100.print("Diff: ");
      Enes100.print(diff);
      Enes100.print("  Strength: ");
      Enes100.println(strength);
      */

    // turns OSV in nearest direction towards assigned heading by passing the 
    // calculated power differential to the left or right motors
    if( diff > 0){
     // Enes100.println("I'm going Right");
      easyRight(strength);
    } else {
      easyLeft(strength);
    //  Enes100.println("I'm going Left");
    }
    
    Enes100.updateLocation();
}


// turns OSV to a given heading. Used for gross corrections and to turn in place
void facing(double theta){
  while(dizzy(theta)){
    correct(theta);
  }
}


/* checks if current OSV heading is within a given heading plus or minus a given tolerance.
* Returns TRUE if current heading is OUTSIDE of allowable range, False if heading is 
* INSIDE allowable range
*
* Uses this convention so that while<tolerance<>>, if<tolerance<>>, etc can be used in
* other functions to quickly determine if the OSV needs to turn or not.
*/
bool dizzy(double h){
  Enes100.updateLocation();
  double thet = Enes100.location.theta;
  if( thet > h + TOLERANCE || thet < h - TOLERANCE){
    return true;
  }
  return false;
}



// left and right turning functions. Passes a value -225 <= t <= 225 to the appropriate motors
// to allow both smooth and in-place turns.

void easyLeft(int t){
  Tank.setLeftMotorPWM(t);
  Tank.setRightMotorPWM(255);
  delay(DELAY);
}

void easyRight(int t){
  Tank.setLeftMotorPWM(255);
  Tank.setRightMotorPWM(t);
  delay(DELAY);
}



// reports if OSV is in top or bottom of arena, and if OSV is facing forwards or backwards.
void reporter(){
  Enes100.updateLocation(); // retrieves current location and heading
  
  // report "Top" if current Y location is above 1 (halfway mark of arena), "Bottom" otherwise
  if(Enes100.location.y > 1){
    Enes100.print("Top, ");
  }else{
    Enes100.print("Bottom, ");
  }
  
  // Reports "Backwards" if OSV heading is greater than pi/2 or less than -pi/2; otherwise "forwards"
  if(Enes100.location.theta > PI/2 || Enes100.location.theta < -PI/2){
    Enes100.println("Backwards");
  }else{
    Enes100.println("Forwards");
  }
}

// absolute value and exponential functions

double absval(double v){
  if(v < 0){
    return -v;
  }
  return v;
}

double power(double base, int exp){
  double result = base;
  for( int i = 1; i < exp; i++){
    result = result * base;
  }
  return result;    
}
