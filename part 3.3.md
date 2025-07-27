# Exercise 3 - Shape Operations
The original code used generic object arrays and many type checking to manage trhe use of many different geometric shapes. This made the code complicated and hard to maintains as things had to change drastically every time a new shape is added. Finally the area and boundary calculations were done with a repeated logic making the code less clean.

The second and improved version uses an interface instead called `Shape` that later then all geometric shapes implement, and so each of them knows already how to compute its own area and boundaries. meanin that the main program needs not to know the unique details of each shape, allowing so for polymorhpism. makign the code easier to read, mantain and extend.



```java
package fi.utu.tech.exercise3;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class Exercise3{
    public Exercise3() {
        System.out.println("Exercise 3");
    }
    record Point(int x, int y){}
    static class Boundary{
        final Point topRight;
        final Point bottomLeft;
        Boundary(Point topRight, Point bottomLeft){
            this.topRight=topRight;
            this.bottomLeft=bottomLeft;
        }

        Boundary intersect(Boundary other){
            int x1=Math.min(this.topRight.x,other.topRight.x);
            int y1=Math.min(this.topRight.y,other.topRight.y);
            int x2=Math.max(this.bottomLeft.x,other.bottomLeft.x);
            int y2=Math.max(this.bottomLeft.y,other.bottomLeft.y);
            return new Boundary(new Point(x1, y1), new Point(x2, y2));
        }
    }
    interface Shape{
        double area();
        Boundary boundaries();
    }
    static class Triangle implements Shape{
        Point p1, p2, p3;
        Triangle(Point p1, Point p2, Point p3){
            this.p1=p1;
            this.p2=p2;
            this.p3=p3;
        }
        @Override
        public double area(){
            int maxX=Math.max(p1.x,Math.max(p2.x,p3.x));
            int maxY=Math.max(p1.y,Math.max(p2.y,p3.y));
            int minX=Math.min(p1.x,Math.min(p2.x,p3.x));
            int minY=Math.min(p1.y,Math.min(p2.y,p3.y));
            return (maxX - minX)*(maxY-minY)/2.0;
        }
        @Override
        public Boundary boundaries(){
            int maxX = Math.max(p1.x,Math.max(p2.x, p3.x));
            int maxY = Math.max(p1.y,Math.max(p2.y, p3.y));
            int minX = Math.min(p1.x,Math.min(p2.x, p3.x));
            int minY = Math.min(p1.y,Math.min(p2.y, p3.y));
            return new Boundary(new Point(maxX, maxY), new Point(minX, minY));
        }
    }
    static class Quadrilateral implements Shape{
        Point p1, p2, p3, p4;
        Quadrilateral(Point p1, Point p2, Point p3, Point p4){
            this.p1=p1;
            this.p2=p2;
            this.p3=p3;
            this.p4=p4;
        }
        @Override
        public double area(){
            int maxX=Math.max(Math.max(p1.x, p2.x),Math.max(p3.x, p4.x));
            int maxY=Math.max(Math.max(p1.y, p2.y),Math.max(p3.y, p4.y));
            int minX=Math.min(Math.min(p1.x, p2.x),Math.min(p3.x, p4.x));
            int minY=Math.min(Math.min(p1.y, p2.y),Math.min(p3.y, p4.y));
            return (maxX - minX)*(maxY - minY);
        }
        @Override
        public Boundary boundaries(){
            int maxX=Math.max(Math.max(p1.x, p2.x),Math.max(p3.x, p4.x));
            int maxY=Math.max(Math.max(p1.y, p2.y),Math.max(p3.y, p4.y));
            int minX=Math.min(Math.min(p1.x, p2.x),Math.min(p3.x, p4.x));
            int minY=Math.min(Math.min(p1.y, p2.y),Math.min(p3.y, p4.y));
            return new Boundary(new Point(maxX, maxY), new Point(minX, minY));
        }
    }
    static class Circle implements Shape{
        Point center;
        Point perimeter;
        Circle(Point center, Point perimeter){
            this.center=center;
            this.perimeter=perimeter;
        }
        @Override
        public double area(){
            int dx=perimeter.x-center.x;
            int dy=perimeter.y-center.y;
            double radiusSquared=dx*dx+dy*dy;
            return Math.PI*radiusSquared;
        }
        @Override
        public Boundary boundaries(){
            int dx=Math.abs(perimeter.x-center.x);
            int dy=Math.abs(perimeter.y-center.y);
            Point topRight=new Point(center.x+dx, center.y+dy);
            Point bottomLeft = new Point(center.x-dx, center.y-dy);
            return new Boundary(topRight,bottomLeft);
        }
    }
    static String readType(){
        System.out.print("Enter the pattern type (triangle, quadrilateral, circle): ");
        return new Scanner(System.in).next().toLowerCase();
    }
    static Point readPoint(){
        Scanner scanner=new Scanner(System.in);
        System.out.print("Enter the x-coordinate of the point: ");
        int x=scanner.nextInt();
        System.out.print("Enter the y-coordinate of the point: ");
        int y=scanner.nextInt();
        return new Point(x, y);
    }
    static Shape readShape() throws Exception{
        String type = readType();
        switch (type){
            case "triangle":
                Point t1=readPoint();
                Point t2=readPoint();
                Point t3=readPoint();
                return new Triangle(t1, t2, t3);
            case "quadrilateral":
                Point q1=readPoint();
                Point q2=readPoint();
                Point q3=readPoint();
                Point q4=readPoint();
                return new Quadrilateral(q1, q2, q3, q4);
            case "circle":
                Point center=readPoint();
                Point perimeter=readPoint();
                return new Circle(center, perimeter);
            default:
                throw new Exception("Unsupported shape type: " + type);
        }
    }
    public static void main(String[] args) throws Exception{
        System.out.println("A circle is defined by a center and a perimeter point, others by corner points.");
        List<Shape> shapes=new ArrayList<>();
        System.out.print("How many shapes do you want to enter? ");
        Scanner sc=new Scanner(System.in);
        int n=sc.nextInt();
        for (int i=0; i<n; i++) {
            System.out.printf("Reading shape #%d:\n", i + 1);
            Shape shape=readShape();
            shapes.add(shape);
        }

        double totalArea=0;
        for (Shape shape: shapes){
            totalArea+=shape.area();
        }
        System.out.printf("Sum of area covered by the patterns: %2f\n\n", totalArea);

        if (shapes.size()>0) {
            Boundary commonBoundary=shapes.get(0).boundaries();
            for (int i=1; i<shapes.size(); i++) {
                commonBoundary=commonBoundary.intersect(shapes.get(i).boundaries());
            }
            System.out.printf("The common boundaries of the patterns:\n" +"(%d, %d) x (%d, %d)\n",
                    commonBoundary.bottomLeft.x, commonBoundary.bottomLeft.y,
                    commonBoundary.topRight.x, commonBoundary.topRight.y);
        } else {
            System.out.println("No shapes entered, no boundaries to show.");
        }
    }
}