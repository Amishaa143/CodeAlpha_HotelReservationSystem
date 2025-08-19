// Hotel Reservtaion Task
import java.io.*;
import java.util.*;

class Room implements Serializable {
    int roomId;
    String category;
    boolean isBooked;

    public Room(int roomId, String category) {
        this.roomId = roomId;
        this.category = category;
        this.isBooked = false;
    }

    public void bookRoom() {
        isBooked = true;
    }

    public void cancelBooking() {
        isBooked = false;
    }

    public String toString() {
        return "Room ID: " + roomId + ", Category: " + category + ", Status: " + (isBooked ? "Booked" : "Available");
    }
}

class Reservation implements Serializable {
    int reservationId;
    String customerName;
    Room room;
    boolean isPaid;

    public Reservation(int reservationId, String customerName, Room room, boolean isPaid) {
        this.reservationId = reservationId;
        this.customerName = customerName;
        this.room = room;
        this.isPaid = isPaid;
    }

    public String toString() {
        return "Reservation ID: " + reservationId + ", Customer: " + customerName + ", " + room.toString() + ", Payment: " + (isPaid ? "Completed" : "Pending");
    }
}

public class task{
    static Scanner sc = new Scanner(System.in);
    static List<Room> rooms = new ArrayList<>();
    static List<Reservation> reservations = new ArrayList<>();
    static int reservationCounter = 1;
    static final String RES_FILE = "reservations.dat";
    static final String ROOM_FILE = "rooms.dat";

    public static void main(String[] args) {
        loadRooms();
        loadReservations();

        while (true) {
            System.out.println("\n===== Hotel Reservation System =====");
            System.out.println("1. View Rooms");
            System.out.println("2. Book Room");
            System.out.println("3. Cancel Reservation");
            System.out.println("4. View Reservations");
            System.out.println("5. Add New Room");
            System.out.println("6. Exit");
            System.out.print("Enter your choice: ");

            int choice = sc.nextInt();
            sc.nextLine();

            switch (choice) {
                case 1 -> viewRooms();
                case 2 -> bookRoom();
                case 3 -> cancelReservation();
                case 4 -> viewReservations();
                case 5 -> addRoom();
                case 6 -> {
                    saveRooms();
                    saveReservations();
                    System.out.println("Exiting... Data saved. Thank you!");
                    return;
                }
                default -> System.out.println("Invalid choice. Try again!");
            }
        }
    }

    private static void viewRooms() {
        System.out.println("\n--- Available Rooms ---");
        for (Room room : rooms) {
            System.out.println(room);
        }
    }

    private static void bookRoom() {
        System.out.print("Enter your name: ");
        String name = sc.nextLine();
        System.out.print("Enter Room ID to book: ");
        int roomId = sc.nextInt();

        for (Room room : rooms) {
            if (room.roomId == roomId && !room.isBooked) {
                System.out.print("Enter payment amount (dummy payment simulation): ");
                double payment = sc.nextDouble();
                boolean isPaid = payment > 0;
                room.bookRoom();
                Reservation res = new Reservation(reservationCounter++, name, room, isPaid);
                reservations.add(res);
                saveReservations();
                saveRooms();
                System.out.println("Booking Successful! " + res);
                return;
            }
        }
        System.out.println("Room not available or invalid Room ID!");
    }

    private static void cancelReservation() {
        System.out.print("Enter Reservation ID to cancel: ");
        int resId = sc.nextInt();

        Iterator<Reservation> it = reservations.iterator();
        while (it.hasNext()) {
            Reservation res = it.next();
            if (res.reservationId == resId) {
                res.room.cancelBooking();
                it.remove();
                saveReservations();
                saveRooms();
                System.out.println("Reservation Cancelled Successfully!");
                return;
            }
        }
        System.out.println("Invalid Reservation ID!");
    }

    private static void viewReservations() {
        System.out.println("\n--- Reservations ---");
        if (reservations.isEmpty()) {
            System.out.println("No reservations found.");
        } else {
            for (Reservation res : reservations) {
                System.out.println(res);
            }
        }
    }

    private static void addRoom() {
        System.out.print("Enter Room ID: ");
        int id = sc.nextInt();
        sc.nextLine();
        System.out.print("Enter Room Category (Standard/Deluxe/Suite): ");
        String category = sc.nextLine();
        rooms.add(new Room(id, category));
        saveRooms();
        System.out.println("Room added successfully!");
    }

    private static void saveReservations() {
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(RES_FILE))) {
            oos.writeObject(reservations);
        } catch (IOException e) {
            System.out.println("Error saving reservations: " + e.getMessage());
        }
    }

    private static void loadReservations() {
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(RES_FILE))) {
            reservations = (List<Reservation>) ois.readObject();
            for (Reservation res : reservations) {
                res.room.bookRoom();
            }
            reservationCounter = reservations.size() + 1;
        } catch (FileNotFoundException e) {
            System.out.println("No saved reservations found.");
        } catch (IOException | ClassNotFoundException e) {
            System.out.println("Error loading reservations: " + e.getMessage());
        }
    }

    private static void saveRooms() {
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(ROOM_FILE))) {
            oos.writeObject(rooms);
        } catch (IOException e) {
            System.out.println("Error saving rooms: " + e.getMessage());
        }
    }

    private static void loadRooms() {
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(ROOM_FILE))) {
            rooms = (List<Room>) ois.readObject();
        } catch (FileNotFoundException e) {
            System.out.println("No saved rooms found. Initializing default rooms.");
            initializeDefaultRooms();
            saveRooms();
        } catch (IOException | ClassNotFoundException e) {
            System.out.println("Error loading rooms: " + e.getMessage());
        }
    }

    private static void initializeDefaultRooms() {
        rooms.add(new Room(101, "Standard"));
        rooms.add(new Room(102, "Standard"));
        rooms.add(new Room(201, "Deluxe"));
        rooms.add(new Room(202, "Deluxe"));
        rooms.add(new Room(301, "Suite"));
    }
}
