import javax.imageio.ImageIO;
import javax.swing.*;
import java.awt.*;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;

public class XRayAnalysis extends JFrame {

    private JPanel xrayPanel;
    private JPanel diffPanel;
    private JTextArea resultArea;

    private BufferedImage xrayImage;
    private BufferedImage crackDetectedImage;

    private boolean isXRayLoaded = false;

    public XRayAnalysis() {
        setupUI();
    }

    private void setupUI() {
        setTitle("X-ray Image Analysis");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLayout(new BorderLayout());
        setSize(1200, 600);

        xrayPanel = new JPanel(new BorderLayout());
        diffPanel = new JPanel(new BorderLayout());
        resultArea = new JTextArea("Load an X-ray image to begin analysis.");
        resultArea.setWrapStyleWord(true);
        resultArea.setLineWrap(true);
        resultArea.setEditable(false);

        JScrollPane xrayScrollPane = new JScrollPane(xrayPanel);
        JScrollPane diffScrollPane = new JScrollPane(diffPanel);
        JScrollPane resultScrollPane = new JScrollPane(resultArea);

        JButton chooseButton = new JButton("Choose X-ray Image");
        JButton highlightButton = new JButton("Highlight Cracks");

        JPanel buttonPanel = new JPanel();
        buttonPanel.add(chooseButton);
        buttonPanel.add(highlightButton);

        JSplitPane splitPane = new JSplitPane(JSplitPane.HORIZONTAL_SPLIT, xrayScrollPane, diffScrollPane);
        add(splitPane, BorderLayout.CENTER);
        add(resultScrollPane, BorderLayout.SOUTH);
        add(buttonPanel, BorderLayout.NORTH);

        chooseButton.addActionListener(e -> chooseImage());
        highlightButton.addActionListener(e -> highlightCracks());

        setLocationRelativeTo(null);
        setVisible(true);
    }

    private void chooseImage() {
        JFileChooser fileChooser = new JFileChooser();
        int result = fileChooser.showOpenDialog(this);

        if (result == JFileChooser.APPROVE_OPTION) {
            File selectedFile = fileChooser.getSelectedFile();
            xrayImage = readImage(selectedFile);

            if (xrayImage != null) {
                displayImage("Original X-ray", xrayImage, xrayPanel);
                crackDetectedImage = detectCracks(xrayImage);
                displayImage("Cracks Detected", crackDetectedImage, diffPanel);
                boolean isHealthyXRay = !hasCracks(crackDetectedImage);
                displayHealthStatus(isHealthyXRay);
                isXRayLoaded = true;

                if (isHealthyXRay) {
                    resultArea.setText("X-ray analysis completed. No cracks detected.");
                } else {
                    resultArea.setText("X-ray analysis completed. Cracks detected.");
                }
            } else {
                JOptionPane.showMessageDialog(this, "Unable to read the image. Please select a valid file.");
            }
        }
    }

    private BufferedImage readImage(File file) {
        try {
            return ImageIO.read(file);
        } catch (IOException e) {
            System.out.println("An error occurred while reading the image: " + e.getMessage());
            return null;
        }
    }

    private BufferedImage detectCracks(BufferedImage xrayImage) {
        BufferedImage simulatedCracks = new BufferedImage(xrayImage.getWidth(), xrayImage.getHeight(), BufferedImage.TYPE_INT_ARGB);
        Graphics g = simulatedCracks.getGraphics();
        g.drawImage(xrayImage, 0, 0, null);
        g.dispose();

        Graphics2D g2d = simulatedCracks.createGraphics();
        g2d.setStroke(new BasicStroke(2));
        g2d.setColor(Color.RED);

        int numCracks = 10;
        for (int i = 0; i < numCracks; i++) {
            int startX = (int) (Math.random() * xrayImage.getWidth());
            int startY = (int) (Math.random() * xrayImage.getHeight());
            int endX = (int) (Math.random() * xrayImage.getWidth());
            int endY = (int) (Math.random() * xrayImage.getHeight());

            g2d.drawLine(startX, startY, endX, endY);
        }

        g2d.dispose();
        return simulatedCracks;
    }

    private boolean hasCracks(BufferedImage image) {
        int redThreshold = 150; // Adjust this threshold as needed
        int redCount = 0;
        int totalPixels = image.getWidth() * image.getHeight();

        for (int x = 0; x < image.getWidth(); x++) {
            for (int y = 0; y < image.getHeight(); y++) {
                int pixel = image.getRGB(x, y);
                Color color = new Color(pixel);

                if (color.getRed() > redThreshold) {
                    redCount++;
                }
            }
        }

        double redPercentage = (double) redCount / totalPixels;
        return redPercentage > 0.01; // Set the threshold for what constitutes cracks
    }

    private void displayImage(String windowName, BufferedImage image, JPanel panel) {
        JLabel label = new JLabel(new ImageIcon(image));
        panel.removeAll();
        panel.add(label);
        panel.revalidate();
        panel.repaint();
    }

    private void highlightCracks() {
        if (crackDetectedImage != null) {
            BufferedImage highlightedCracks = new BufferedImage(crackDetectedImage.getWidth(),
                    crackDetectedImage.getHeight(), BufferedImage.TYPE_INT_ARGB);

            for (int x = 0; x < crackDetectedImage.getWidth(); x++) {
                for (int y = 0; y < crackDetectedImage.getHeight(); y++) {
                    int pixel = crackDetectedImage.getRGB(x, y);
                    int redComponent = (pixel >> 16) & 0xFF;

                    if (redComponent > 100) {
                        highlightedCracks.setRGB(x, y, Color.YELLOW.getRGB());
                    } else {
                        highlightedCracks.setRGB(x, y, pixel);
                    }
                }
            }

            displayImage("Highlighted Cracks", highlightedCracks, diffPanel);
        }
    }

    private void displayHealthStatus(boolean isHealthy) {
        diffPanel.removeAll();
        diffPanel.setLayout(new BoxLayout(diffPanel, BoxLayout.Y_AXIS));

        JLabel healthStatusLabel = new JLabel("Health Status: " + (isHealthy ? "Healthy" : "Unhealthy"));
        diffPanel.add(healthStatusLabel);

        diffPanel.revalidate();
        diffPanel.repaint();
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(XRayAnalysis::new);
    }
}
