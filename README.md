
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.awt.geom.*;
import java.util.*;
import java.util.List;

public class HalloweenSurprise extends JPanel {

    private final javax.swing.Timer timer;
    private final Random random = new Random();

    private double swing = 0;
    private double night = 0;
    private double open = 0;
    private int tick = 0;

    private boolean clicked = false;
    private boolean showMessage = false;

    private final List<Particle> particles = new ArrayList<>();

    private static final int W = 900;
    private static final int H = 650;

    static class Particle {
        double x, y, vx, vy, life, size;
        Color color;
        String symbol;

        Particle(double x, double y, Color color, String symbol) {
            this.x = x;
            this.y = y;
            this.color = color;
            this.symbol = symbol;
            this.size = 5 + Math.random() * 13;
            this.life = 1;
            double angle = Math.random() * Math.PI * 2;
            double speed = 1 + Math.random() * 5;
            vx = Math.cos(angle) * speed;
            vy = Math.sin(angle) * speed - 2;
        }

        void update() {
            x += vx;
            y += vy;
            vy += 0.045;
            vx *= 0.985;
            life -= 0.012;
        }
    }

    public HalloweenSurprise() {
        setPreferredSize(new Dimension(W, H));
        setBackground(Color.BLACK);
        setCursor(Cursor.getPredefinedCursor(Cursor.HAND_CURSOR));

        addMouseListener(new MouseAdapter() {
            @Override
            public void mouseClicked(MouseEvent e) {
                if (!clicked && e.getX() >= 320 && e.getX() <= 580
                        && e.getY() >= 310 && e.getY() <= 550) {
                    clicked = true;
                    showMessage = true;
                    open = 0;
                    burst(450, 390, 130);
                    burst(450, 350, 80);
                }
            }
        });

        timer = new javax.swing.Timer(16, e -> {
            tick++;
            swing = Math.sin(tick * 0.07) * 0.09;

            if (clicked) {
                night = Math.min(1, night + 0.006);
                open = Math.min(1, open + 0.025);

                if (tick % 5 == 0) {
                    burst(
                        350 + random.nextInt(200),
                        180 + random.nextInt(220),
                        3
                    );
                }
            }

            for (Particle p : particles) {
                p.update();
            }
            particles.removeIf(p -> p.life <= 0);

            repaint();
        });
        timer.start();
    }

    private Color mix(Color a, Color b, double t) {
        t = Math.max(0, Math.min(1, t));
        return new Color(
            (int)(a.getRed() * (1-t) + b.getRed() * t),
            (int)(a.getGreen() * (1-t) + b.getGreen() * t),
            (int)(a.getBlue() * (1-t) + b.getBlue() * t)
        );
    }

    private void burst(int x, int y, int count) {
        Color[] colors = {
            new Color(255, 90, 60),
            new Color(255, 190, 60),
            new Color(255, 100, 190),
            new Color(180, 130, 255),
            new Color(120, 230, 255),
            Color.WHITE
        };

        for (int i = 0; i < count; i++) {
            Color c = colors[random.nextInt(colors.length)];
            String[] symbols = {"*", "✦", "♥", "•"};
            String s = symbols[random.nextInt(symbols.length)];
            particles.add(new Particle(x, y, c, s));
        }
    }

    @Override
    protected void paintComponent(Graphics graphics) {
        super.paintComponent(graphics);

        Graphics2D g = (Graphics2D) graphics.create();
        g.setRenderingHint(
            RenderingHints.KEY_ANTIALIASING,
            RenderingHints.VALUE_ANTIALIAS_ON
        );

        // Adapt to the actual window size
        g.scale(getWidth() / (double) W, getHeight() / (double) H);

        drawRoom(g);
        drawWindow(g);
        drawDecorations(g);
        drawGift(g);
        drawParticles(g);

        if (showMessage) {
            drawMessage(g);
        } else {
            drawInstructions(g);
        }

        g.dispose();
    }

    private void drawRoom(Graphics2D g) {
        // Cozy room gradually changes into a mysterious night scene
        g.setPaint(new GradientPaint(
            0, 0, mix(new Color(85, 48, 70),
                      new Color(10, 12, 39), night),
            0, 650, mix(new Color(155, 91, 63),
                        new Color(28, 23, 53), night)
        ));
        g.fillRect(0, 0, W, H);

        // Wall decorations
        g.setColor(new Color(255, 190, 115, 25));
        for (int x = 30; x < W; x += 55) {
            g.drawLine(x, 0, x, 470);
        }

        // Floor
        g.setPaint(new GradientPaint(
            0, 480, new Color(95, 53, 49),
            0, 650, new Color(35, 25, 42)
        ));
        g.fillRect(0, 480, W, 170);

        g.setColor(new Color(255, 210, 150, 35));
        for (int y = 500; y < 650; y += 32) {
            g.drawLine(0, y, W, y);
        }

        // Warm wall lamp
        g.setColor(new Color(255, 183, 80, 50));
        g.fillOval(50, 100, 150, 200);
        g.setColor(new Color(255, 190, 105));
        g.fillOval(100, 125, 48, 70);
        g.setColor(new Color(255, 225, 165));
        g.fillOval(110, 135, 28, 45);

        // Rug
        g.setColor(new Color(95, 31, 56));
        g.fillRoundRect(190, 465, 520, 150, 45, 45);
        g.setColor(new Color(220, 139, 91, 130));
        g.setStroke(new BasicStroke(2));
        g.drawRoundRect(205, 480, 490, 120, 35, 35);
    }

    private void drawWindow(Graphics2D g) {
        int x = 570, y = 65, w = 270, h = 320;

        // Window frame
        g.setColor(new Color(48, 28, 42));
        g.fillRoundRect(x - 12, y - 12, w + 24, h + 24, 18, 18);

        // Sky changes from sunset to midnight
        g.setPaint(new GradientPaint(
            x, y, mix(new Color(235, 105, 81),
                      new Color(5, 9, 36), night),
            x, y + h, mix(new Color(91, 56, 112),
                          new Color(17, 25, 65), night)
        ));
        g.fillRect(x, y, w, h);

        // Stars appear as the sky becomes darker
        if (night > 0.15) {
            for (int i = 0; i < 65; i++) {
                int sx = x + (i * 47 + 17) % w;
                int sy = y + (i * 31 + 19) % 210;
                int alpha = (int)(255 * night);
                g.setColor(new Color(255, 240, 200, alpha));
                int r = i % 5 == 0 ? 3 : 1;
                g.fillOval(sx, sy, r, r);
            }
        }

        // Moon
        int moonX = x + 180;
        int moonY = y + 40;
        g.setColor(new Color(255, 241, 196));
        g.fillOval(moonX, moonY, 48, 48);
        g.setColor(mix(new Color(235, 105, 81),
                       new Color(5, 9, 36), night));
        g.fillOval(moonX - 13, moonY - 7, 43, 43);

        // Distant Halloween silhouettes
        g.setColor(new Color(23, 23, 48));
        g.fillRect(x, y + 255, w, 65);
        int[] hx = {x, x+30, x+55, x+75, x+100, x+130,
                    x+160, x+190, x+220, x+245, x+w};
        int[] hy = {y+275, y+260, y+275, y+250, y+275,
                    y+258, y+278, y+255, y+270, y+250, y+270};
        g.fillPolygon(hx, hy, hx.length);

        // Window crossbars
        g.setColor(new Color(70, 39, 48));
        g.setStroke(new BasicStroke(9));
        g.drawLine(x + w/2, y, x + w/2, y + h);
        g.drawLine(x, y + 170, x + w, y + 170);

        // Curtains
        g.setColor(new Color(104, 34, 60));
        g.fillPolygon(
            new int[]{x-20, x+48, x+35, x+10, x-20},
            new int[]{y-10, y-10, y+225, y+190, y+270}, 5);
        g.fillPolygon(
            new int[]{x+w-48, x+w+20, x+w+20,
                      x+w-5, x+w-35},
            new int[]{y-10, y-10, y+270, y+190, y+225}, 5);
    }

    private void drawPumpkin(Graphics2D g, int x, int y, int size) {
        g.setColor(new Color(240, 111, 34));
        g.fillOval(x, y + 5, size, size * 3 / 4);
        g.fillOval(x + size/4, y, size/2, size * 3 / 4);
        g.fillOval(x + size/2, y + 5, size/2, size * 3 / 4);

        g.setColor(new Color(67, 108, 49));
        g.fillRoundRect(x + size/2 - 4, y - 8, 9, 18, 5, 5);

        // Jack-o'-lantern face
        g.setColor(new Color(75, 31, 31));
        int eye = Math.max(4, size / 8);
        g.fillPolygon(
            new int[]{x+size/4, x+size/4+eye, x+size/4+eye/2},
            new int[]{y+size/3, y+size/3, y+size/3+eye}, 3);
        g.fillPolygon(
            new int[]{x+size*2/3, x+size*2/3+eye, x+size*2/3+eye/2},
            new int[]{y+size/3, y+size/3, y+size/3+eye}, 3);
        g.drawArc(x+size/4, y+size/2, size/2, size/4, 180, 180);
    }

    private void drawDecorations(Graphics2D g) {
        // Hanging bats
        g.setColor(new Color(35, 22, 39));
        for (int i = 0; i < 3; i++) {
            int x = 280 + i * 70;
            int y = 95 + (i % 2) * 24;
            g.drawLine(x, 0, x, y);
            g.fillOval(x-10, y, 20, 12);
            g.fillPolygon(
                new int[]{x-10, x-24, x-18, x-5},
                new int[]{y+4, y+15, y+15, y+9}, 4);
            g.fillPolygon(
                new int[]{x+10, x+24, x+18, x+5},
                new int[]{y+4, y+15, y+15, y+9}, 4);
        }

        drawPumpkin(g, 55, 510, 72);
        drawPumpkin(g, 770, 500, 65);

        // A tiny ghost
        g.setColor(new Color(245, 235, 224, 220));
        g.fillOval(740, 410, 35, 35);
        g.fillRoundRect(740, 425, 35, 38, 12, 12);
        g.setColor(new Color(39, 31, 51));
        g.fillOval(748, 423, 5, 7);
        g.fillOval(762, 423, 5, 7);
    }

    private void drawGift(Graphics2D g) {
        Graphics2D gift = (Graphics2D) g.create();

        double angle = clicked ? swing * (1 - open) : swing;
        gift.rotate(angle, 450, 430);

        // Shadow
        gift.setColor(new Color(0, 0, 0, 75));
        gift.fillOval(340, 485, 220, 35);

        // Gift body
        gift.setPaint(new GradientPaint(
            350, 380, new Color(171, 29, 76),
            530, 490, new Color(90, 16, 54)
        ));
        gift.fillRoundRect(355, 385, 190, 120, 12, 12);

        // Gift ribbons
        gift.setColor(new Color(255, 205, 106));
        gift.fillRect(433, 385, 32, 120);
        gift.fillRect(355, 425, 190, 25);

        // Bow
        gift.setStroke(new BasicStroke(7));
        gift.drawArc(407, 345, 48, 45, 0, 270);
        gift.drawArc(444, 345, 48, 45, -90, 270);
        gift.fillOval(438, 365, 18, 18);

        // Lid opens after clicking
        if (clicked) {
            double lift = open * 115;
            gift.setColor(new Color(125, 15, 55));
            gift.fillRoundRect(345, (int)(365 - lift),
                               210, 28, 8, 8);
            gift.setColor(new Color(255, 205, 106));
            gift.fillRect(433, (int)(365 - lift), 32, 28);

            // Magical light coming from inside
            if (open > 0.25) {
                int alpha = (int)(130 * Math.sin(open * Math.PI));
                gift.setColor(new Color(255, 223, 125,
                                        Math.max(0, alpha)));
                gift.fillOval(375, 330, 150, 130);
            }
        } else {
            // Label
            gift.setColor(new Color(255, 239, 192));
            gift.setFont(new Font("Serif", Font.BOLD, 15));
            gift.drawString("FOR YOU ♥", 395, 470);
        }

        gift.dispose();
    }

    private void drawParticles(Graphics2D g) {
        for (Particle p : particles) {
            int alpha = (int)(255 * Math.max(0, Math.min(1, p.life)));
            g.setColor(new Color(
                p.color.getRed(), p.color.getGreen(),
                p.color.getBlue(), alpha
            ));
            g.setFont(new Font("Serif", Font.BOLD, (int)p.size + 8));
            g.drawString(p.symbol, (int)p.x, (int)p.y);
        }
    }

    private void drawInstructions(Graphics2D g) {
        g.setColor(new Color(255, 239, 210));
        g.setFont(new Font("Serif", Font.BOLD, 25));
        centerText(g, "A Little Halloween Surprise", 575);

        g.setColor(new Color(255, 212, 165));
        g.setFont(new Font("SansSerif", Font.PLAIN, 16));
        centerText(g, "Click the gift to open your surprise ♥", 607);
    }

    private void drawMessage(Graphics2D g) {
        // Dark transparent overlay
        g.setColor(new Color(5, 5, 25, (int)(105 * night)));
        g.fillRect(0, 0, W, H);

        // Glowing message panel
        g.setColor(new Color(25, 15, 45, 215));
        g.fillRoundRect(165, 175, 570, 225, 35, 35);

        g.setColor(new Color(255, 192, 110, 220));
        g.setStroke(new BasicStroke(2));
        g.drawRoundRect(165, 175, 570, 225, 35, 35);

        g.setColor(new Color(255, 170, 205));
        g.setFont(new Font("Serif", Font.BOLD, 39));
        centerText(g, "万圣节快乐呀🎃!", 235);

        g.setColor(new Color(255, 226, 177));
        g.setFont(new Font("SansSerif", Font.PLAIN, 21));
        centerText(g, "至我最爱的人 ♥", 282);

        g.setColor(Color.WHITE);
        g.setFont(new Font("SansSerif", Font.PLAIN, 17));
        centerText(g, "虽然我们离的这么远,", 325);
        centerText(g, "但对我来说你就是全世界最好的人...我一生的挚爱", 354);

        g.setColor(new Color(255, 167, 210));
        g.setFont(new Font("Serif", Font.BOLD, 23));
        centerText(g, "希望我的宝宝身体健康万事顺遂 ♥", 382);

        g.setColor(new Color(255, 220, 150));
        g.setFont(new Font("SansSerif", Font.PLAIN, 14));
        centerText(g, "秦秦宝宝mua! ✨", 435);
    }

    private void centerText(Graphics2D g, String text, int y) {
        FontMetrics fm = g.getFontMetrics();
        int x = (W - fm.stringWidth(text)) / 2;
        g.drawString(text, x, y);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("A Halloween Surprise ♥");
            HalloweenSurprise scene = new HalloweenSurprise();

            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setContentPane(scene);
            frame.pack();
            frame.setMinimumSize(new Dimension(700, 520));
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
