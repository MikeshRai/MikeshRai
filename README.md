import javafx.application.Application;
import javafx.geometry.Pos;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.control.TextField;
import javafx.scene.layout.GridPane;
import javafx.stage.Stage;

public class GUI_Calculator extends Application {

    private TextField inputField;

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        primaryStage.setTitle("Calculator");

        inputField = new TextField();
        inputField.setEditable(false);
        inputField.setAlignment(Pos.CENTER);
        inputField.setStyle("-fx-font-size: 18; -fx-pref-width: 250;");

        GridPane gridPane = new GridPane();
        gridPane.setAlignment(Pos.CENTER);
        gridPane.setHgap(10);
        gridPane.setVgap(10);

        String[] buttonLabels = {
            "7", "8", "9", "/",
            "4", "5", "6", "*",
            "1", "2", "3", "-",
            "0", "C", "=", "+"
        };

        Button[] buttons = new Button[buttonLabels.length];

        for (int i = 0; i < buttonLabels.length; i++) {
            buttons[i] = new Button(buttonLabels[i]);
            buttons[i].setStyle("-fx-font-size: 18; -fx-min-width: 60; -fx-min-height: 60;");
        }

        // Set actions for buttons
        for (int i = 0; i < buttons.length; i++) {
            Button button = buttons[i];
            button.setOnAction(e -> onButtonClicked(button.getText()));
        }

        gridPane.add(inputField, 0, 0, 4, 1);

        int buttonIndex = 0;
        for (int row = 1; row <= 4; row++) {
            for (int col = 0; col < 4; col++) {
                gridPane.add(buttons[buttonIndex], col, row);
                buttonIndex++;
            }
        }

        Scene scene = new Scene(gridPane, 300, 400);
        primaryStage.setScene(scene);

        primaryStage.show();
    }

    private void onButtonClicked(String text) {
        switch (text) {
            case "C":
                inputField.clear();
                break;
            case "=":
                try {
                    String expression = inputField.getText();
                    double result = Calculator.evaluate(expression);
                    inputField.setText(Double.toString(result));
                } catch (Exception ex) {
                    inputField.setText("Error");
                }
                break;
            default:
                inputField.appendText(text);
        }
    }

    public static class Calculator {
        public static double evaluate(String expression) {
            return new Object() {
                int pos = -1, ch;

                void nextChar() {
                    ch = (++pos < expression.length()) ? expression.charAt(pos) : -1;
                }

                boolean eat(int charToEat) {
                    while (ch == ' ') nextChar();
                    if (ch == charToEat) {
                        nextChar();
                        return true;
                    }
                    return false;
                }

                double parse() {
                    nextChar();
                    double x = parseExpression();
                    if (pos < expression.length()) throw new RuntimeException("Unexpected: " + (char) ch);
                    return x;
                }

                double parseExpression() {
                    double x = parseTerm();
                    for (; ; ) {
                        if (eat('+')) x += parseTerm();
                        else if (eat('-')) x -= parseTerm();
                        else return x;
                    }
                }

                double parseTerm() {
                    double x = parseFactor();
                    for (; ; ) {
                        if (eat('*')) x *= parseFactor();
                        else if (eat('/')) x /= parseFactor();
                        else return x;
                    }
                }

                double parseFactor() {
                    if (eat('+')) return parseFactor();
                    if (eat('-')) return -parseFactor();

                    double x;
                    int startPos = this.pos;
                    if (eat('(')) {
                        x = parseExpression();
                        eat(')');
                    } else if ((ch >= '0' && ch <= '9') || ch == '.') {
                        while ((ch >= '0' && ch <= '9') || ch == '.') nextChar();
                        x = Double.parseDouble(expression.substring(startPos, this.pos));
                    } else {
                        throw new RuntimeException("Unexpected: " + (char) ch);
                    }

                    if (eat('^')) x = Math.pow(x, parseFactor());

                    return x;
                }
            }.parse();
        }
    }

    public static void main(String[] args) {
        launch(args);
    }
}
