# Hair-Cutter-Machine
✂️ Electric Hair Cutter Simulator is a Java console app modeling a real hair cutter. It features OOP (encapsulation, inheritance, polymorphism), multithreading with ReentrantLock, blade/battery management, and real-time logging. Learn Java 23 with fun, hands-on device simulation!\

////////////////////////////////// Hair Cutter Class ///////////////////////////

package project;
import java.util.Scanner;

public class HairCutterMain {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        Battery battery = new Battery(100); // Aggregation: Battery created independently
        ElectricHairCutter hairCutter = new ElectricHairCutter("panasonic", battery);

        while (true) {
            System.out.println("\nElectric Hair Cutter Menu:");
            System.out.println("1. Turn ON");
            System.out.println("2. Turn OFF");
            System.out.println("3. Operate");
            System.out.println("4. Adjust Blade Length");
            System.out.println("5. Recharge Battery");
            System.out.println("6. Exit");
            System.out.print("Choose an option: ");
            System.out.println("choose number from(1 to 6)");
            int choice= scanner.nextInt();


            switch (choice) {
                case 1:
                    hairCutter.turnOn();
                    hairCutter.startMonitoring(); // Start monitoring thread
                    hairCutter.startOperation(); // Start operation thread
                    break;
                case 2:
                    hairCutter.turnOff();
                    break;
                case 3:
                    hairCutter.operate();
                    break;
                case 4:
                    System.out.print("Enter blade length (1-10 mm): ");
                    try {
                        int length = Integer.parseInt(scanner.nextLine());
                        hairCutter.adjustBladeLength(length);
                    } catch (NumberFormatException e) {
                        System.out.println("Invalid input. Please enter a number.");
                    }
                    break;
                case 5:
                    battery.recharge();
                    hairCutter.logOperation("Battery recharged to 100%");
                    System.out.println("Battery recharged to 100%");
                    break;
                case 6:
                    System.out.println("Exiting...");
                    scanner.close();
                    System.exit(0);
                default:
                    System.out.println("Invalid option. Try again.");
            }
        }
    }
}

//////////////////////////////// Blade Class ///////////////////////////////

package project;
public class Blade {
    private int sharpnessLevel; // Encapsulation: private attribute

    public Blade() {
        this.sharpnessLevel = 100; // 100% sharpness initially
    }

    // Getter and setter for encapsulation
    public int getSharpnessLevel() {
        return sharpnessLevel;
    }

    public void setSharpnessLevel(int sharpnessLevel) {
        if (sharpnessLevel >= 0 && sharpnessLevel <= 100) {
            this.sharpnessLevel = sharpnessLevel;
        }
    }

    public void useBlade() {
        if (sharpnessLevel > 0) {
            sharpnessLevel -= 10;
            if (sharpnessLevel < 0) sharpnessLevel = 0;
        }
    }
}

/////////////////////////// Appliance Class //////////////////////////////////

package project;
import java.io.FileWriter;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;


public abstract class Appliance {
    protected String brand;
    protected boolean isOn;

    public Appliance(String brand) {
        this.brand = brand;
        this.isOn = false;
    }


    public abstract void operate();

    public void turnOn() {
        isOn = true;
        logOperation("Appliance turned ON");
    }

    public void turnOff() {
        isOn = false;
        logOperation("Appliance turned OFF");
    }

    public boolean isOn() {
        return isOn;
    }

    protected void logOperation(String message) {
        try (FileWriter writer = new FileWriter("hair_cutter_log.txt", true)) {
            String timestamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
            writer.write(timestamp + ": " + message + "\n");
        } catch (IOException e) {
            System.out.println("Error logging operation: " + e.getMessage());
        }
    }
}

//////////////////////////////////////// Motor Class //////////////////////

package project;
public class Motor {
    private int speed; // Encapsulation: private attribute

    public Motor() {
        this.speed = 0;
    }


    public int getSpeed() {
        return speed;
    }

    public void setSpeed(int speed) {
        if (speed >= 0 && speed <= 100) {
            this.speed = speed;
        }
    }

    public void run() {
        if (speed > 0) {
            System.out.println("Motor running at speed: " + speed);
        }
    }
}

////////////////////////////////// Battery Class ////////////////////////////////

package project;
public class Battery {
    private int chargeLevel;

    public Battery(int chargeLevel) {
        this.chargeLevel = chargeLevel;
    }

    public int getChargeLevel() {
        return chargeLevel;
    }

    public void consumeCharge(int amount) {
        chargeLevel -= amount;
        if (chargeLevel < 0) chargeLevel = 0;
    }

    public void recharge() {
        chargeLevel = 100;
    }
}

//////////////////////////////////////// Electric Hair Cutter /////////////////////////////

package project;
import java.util.concurrent.locks.ReentrantLock;

public class ElectricHairCutter extends Appliance {
    private Blade blade; // Composition
    private Motor motor; // Composition
    private Battery battery; // Aggregation
    private int bladeLength; //  adjustable blade length
    private final ReentrantLock lock = new ReentrantLock(); // For thread synchronization

    public ElectricHairCutter(String brand, Battery battery) {
        super(brand);
        this.blade = new Blade();
        this.motor = new Motor();
        this.battery = battery; // Aggregation: Battery exists independently
        this.bladeLength = 1; // Default blade length in mm
    }

    // Polymorphism: Override operate method
    @Override
    public void operate() {
        lock.lock();
        try {
            if (isOn && battery.getChargeLevel() > 0) {
                motor.setSpeed(50); // Set motor speed
                motor.run();
                blade.useBlade();
                battery.consumeCharge(5);
                logOperation("Hair cutter operating, blade length: " + bladeLength + "mm, battery: " + battery.getChargeLevel() + "%");
                System.out.println("Cutting hair with blade length: " + bladeLength + "mm");
            } else {
                System.out.println("Cannot operate: " + (isOn ? "Battery depleted" : "Device is OFF"));
                logOperation("Operation failed: " + (isOn ? "Battery depleted" : "Device is OFF"));
            }
        } finally {
            lock.unlock();
        }
    }

    // Unique feature: Adjust blade length
    public void adjustBladeLength(int length) {
        lock.lock();
        try {
            if (length >= 1 && length <= 10) {
                this.bladeLength = length;
                logOperation("Blade length adjusted to " + length + "mm");
                System.out.println("Blade length set to: " + length + "mm");
            } else {
                System.out.println("Invalid blade length. Must be between 1 and 10 mm.");
            }
        } finally {
            lock.unlock();
        }
    }

    // Thread for monitoring status
    public void startMonitoring() {
        Thread monitorThread = new Thread(() -> {
            while (isOn) {
                lock.lock();
                try {
                    System.out.println("Status - Battery: " + battery.getChargeLevel() + "%, Blade Sharpness: " + blade.getSharpnessLevel() + "%, Motor Speed: " + motor.getSpeed());
                    logOperation("Status - Battery: " + battery.getChargeLevel() + "%, Blade Sharpness: " + blade.getSharpnessLevel() + "%");
                } finally {
                    lock.unlock();
                }
                try {
                    Thread.sleep(5000); // Monitor every 5 seconds
                } catch (InterruptedException e) {
                    logOperation("Monitoring interrupted: " + e.getMessage());
                    Thread.currentThread().interrupt(); // Restore interrupted status
                }
            }
        });
        monitorThread.start();
    }

    // Thread for continuous operation
    public void startOperation() {
        Thread operationThread = new Thread(() -> {
            while (isOn && battery.getChargeLevel() > 0) {
                operate();
                try {
                    Thread.sleep(3000); // Operate every 3 seconds
                } catch (InterruptedException e) {
                    logOperation("Operation interrupted: " + e.getMessage());
                    Thread.currentThread().interrupt(); // Restore interrupted status
                }
            }
        });
        operationThread.start();
    }
}
