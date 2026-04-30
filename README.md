import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.util.ArrayList;

// Welcome Screen
class WelcomeScreen extends JFrame {
    WelcomeScreen() {
        setTitle("Welcome");
        setSize(400, 300);

        JLabel label = new JLabel("Welcome to Smart Calculator", JLabel.CENTER);
        label.setFont(new Font("Arial", Font.BOLD, 22));
        label.setForeground(Color.WHITE);

        getContentPane().setBackground(new Color(30,144,255));
        add(label);

        setLocationRelativeTo(null);
        setVisible(true);

        new Timer(2000, e -> {
            new CalculatorApp();
            dispose();
        }).start();
    }
}

// History Screen
class HistoryScreen extends JFrame {
    HistoryScreen(ArrayList<String> history) {
        setTitle("History");
        setSize(300, 400);

        JTextArea area = new JTextArea();
        area.setEditable(false);

        for(String s: history){
            area.append(s + "\n");
        }

        JButton back = new JButton("Back");
        back.addActionListener(e -> dispose());

        add(new JScrollPane(area), BorderLayout.CENTER);
        add(back, BorderLayout.SOUTH);

        setLocationRelativeTo(null);
        setVisible(true);
    }
}

// Main Calculator
public class CalculatorApp extends JFrame implements ActionListener {

    JTextField textField;
    JLabel messageLabel;

    JButton[] numbers = new JButton[10];
    JButton add, sub, mul, div, equal, clear, dot, historyBtn;

    double num1, num2, result;
    char operator;

    ArrayList<String> history = new ArrayList<>();

    CalculatorApp() {
        setTitle("Smart Calculator");
        setSize(320, 500);
        setLayout(new BorderLayout(10,10));
        getContentPane().setBackground(Color.BLACK);

        // Display
        textField = new JTextField();
        textField.setFont(new Font("Arial", Font.BOLD, 24));
        textField.setBackground(Color.BLACK);
        textField.setForeground(Color.GREEN);
        textField.setHorizontalAlignment(JTextField.RIGHT);
        add(textField, BorderLayout.NORTH);

        // Message
        messageLabel = new JLabel(" ", JLabel.CENTER);
        messageLabel.setForeground(Color.YELLOW);
        add(messageLabel, BorderLayout.SOUTH);

        // Panel
        JPanel panel = new JPanel(new GridLayout(6,4,10,10));
        panel.setBackground(Color.BLACK);

        // Numbers
        for(int i=0;i<10;i++){
            numbers[i] = createNumberButton(""+i);
        }

        // Operators
        add = createOperatorButton("+");
        sub = createOperatorButton("-");
        mul = createOperatorButton("*");
        div = createOperatorButton("/");
        equal = createOperatorButton("=");
        clear = createOperatorButton("C");
        dot = createOperatorButton(".");
        historyBtn = createOperatorButton("H");

        // Layout
        panel.add(numbers[7]); panel.add(numbers[8]); panel.add(numbers[9]); panel.add(div);
        panel.add(numbers[4]); panel.add(numbers[5]); panel.add(numbers[6]); panel.add(mul);
        panel.add(numbers[1]); panel.add(numbers[2]); panel.add(numbers[3]); panel.add(sub);
        panel.add(numbers[0]); panel.add(dot); panel.add(equal); panel.add(add);
        panel.add(clear); panel.add(historyBtn);

        add(panel, BorderLayout.CENTER);

        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        setVisible(true);
    }

    // Number Button Style
    JButton createNumberButton(String text){
        JButton btn = new JButton(text);
        btn.setFont(new Font("Arial", Font.BOLD, 18));
        btn.setBackground(new Color(60,60,60));
        btn.setForeground(Color.WHITE);
        btn.setFocusPainted(false);

        btn.addMouseListener(new MouseAdapter() {
            public void mousePressed(MouseEvent e) {
                btn.setBackground(new Color(120,120,120));
            }
            public void mouseReleased(MouseEvent e) {
                new Timer(100, ev -> btn.setBackground(new Color(60,60,60))).start();
            }
        });

        btn.addActionListener(this);
        return btn;
    }

    // Operator Button Style
    JButton createOperatorButton(String text){
        JButton btn = new JButton(text);
        btn.setFont(new Font("Arial", Font.BOLD, 18));
        btn.setBackground(new Color(255,140,0));
        btn.setForeground(Color.WHITE);
        btn.setFocusPainted(false);

        btn.addMouseListener(new MouseAdapter() {
            public void mousePressed(MouseEvent e) {
                btn.setBackground(Color.RED);
            }
            public void mouseReleased(MouseEvent e) {
                new Timer(100, ev -> btn.setBackground(new Color(255,140,0))).start();
            }
        });

        btn.addActionListener(this);
        return btn;
    }

    // Voice
    void speak(String text){
        try{
            Runtime.getRuntime().exec(
                "powershell -Command \"Add-Type –AssemblyName System.Speech; " +
                "(New-Object System.Speech.Synthesis.SpeechSynthesizer).Speak('" + text + "');\""
            );
        }catch(Exception e){}
    }

    public void actionPerformed(ActionEvent e){
        try{
            for(int i=0;i<10;i++){
                if(e.getSource()==numbers[i]){
                    textField.setText(textField.getText()+i);
                    speak(""+i);
                }
            }

            if(e.getSource()==dot){
                textField.setText(textField.getText()+".");
            }

            if(e.getSource()==add){
                num1 = Double.parseDouble(textField.getText());
                operator='+';
                textField.setText(num1 + " + ");
            }

            if(e.getSource()==sub){
                num1 = Double.parseDouble(textField.getText());
                operator='-';
                textField.setText(num1 + " - ");
            }

            if(e.getSource()==mul){
                num1 = Double.parseDouble(textField.getText());
                operator='*';
                textField.setText(num1 + " * ");
            }

            if(e.getSource()==div){
                num1 = Double.parseDouble(textField.getText());
                operator='/';
                textField.setText(num1 + " / ");
            }

            if(e.getSource()==equal){
                String[] parts = textField.getText().split(" ");
                if(parts.length < 3) throw new Exception("Incomplete input");

                num2 = Double.parseDouble(parts[2]);

                if(operator=='/' && num2==0)
                    throw new ArithmeticException("Cannot divide by zero");

                switch(operator){
                    case '+': result=num1+num2; break;
                    case '-': result=num1-num2; break;
                    case '*': result=num1*num2; break;
                    case '/': result=num1/num2; break;
                }

                String output = "You got the answer: " + result;
                textField.setText(output);
                messageLabel.setText("Calculation successful ✅");

                history.add(num1+" "+operator+" "+num2+" = "+result);
                speak("The answer is " + result);
            }

            if(e.getSource()==clear){
                textField.setText("");
                messageLabel.setText("Cleared");
            }

            if(e.getSource()==historyBtn){
                new HistoryScreen(history);
            }

        }catch(Exception ex){
            messageLabel.setText("❗ " + ex.getMessage());
            speak("error");
        }
    }

    public static void main(String[] args){
        new WelcomeScreen();
    }
}

