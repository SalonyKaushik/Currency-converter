# Currency-converter
i develop a currency converter by using java programming
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

public class CurrencyConverter {
    private static final String API_URL = "https://exchangerate-api.com/v4/latest/"; // Replace with your API's URL and key
    private Map<String, Double> exchangeRates;

    public CurrencyConverter(String baseCurrency) {
        this.exchangeRates = fetchExchangeRates(baseCurrency);
    }

    public double convert(String targetCurrency, double amount) {
        Double rate = exchangeRates.get(targetCurrency);
        return rate != null ? amount * rate : 0.0;
    }

    private Map<String, Double> fetchExchangeRates(String baseCurrency) {
        Map<String, Double> rates = new HashMap<>();
        try {
            URL url = new URL(API_URL + baseCurrency); // endpoint for fetching rates
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setRequestMethod("GET");
            BufferedReader reader = new BufferedReader(new InputStreamReader(conn.getInputStream()));
            StringBuilder json = new StringBuilder();
            String line;

            while ((line = reader.readLine()) != null) {
                json.append(line);
            }
            reader.close();

            // Assuming the API returns a JSON like: {"rates": {"USD": 1.0, "EUR": 0.85, ...}}
            String response = json.toString();
            String ratesJson = response.substring(response.indexOf("\"rates\": {") + 9, response.indexOf("}"));
            String[] pairs = ratesJson.split(",");
            for (String pair : pairs) {
                String[] keyValue = pair.split(":");
                String currency = keyValue[0].replaceAll("\"", "").trim();
                double rate = Double.parseDouble(keyValue[1].trim());
                rates.put(currency, rate);
            }
        } catch (IOException e) {
            System.out.println("Error fetching exchange rates: " + e.getMessage());
        }
        return rates;
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter base currency (e.g., USD, EUR): ");
        String baseCurrency = scanner.nextLine().toUpperCase();

        CurrencyConverter converter = new CurrencyConverter(baseCurrency);

        System.out.print("Enter target currency (e.g., USD, EUR): ");
        String targetCurrency = scanner.nextLine().toUpperCase();

        System.out.print("Enter amount to convert: ");
        double amount = scanner.nextDouble();

        double convertedAmount = converter.convert(targetCurrency, amount);

        if (convertedAmount > 0) {
            System.out.printf("%.2f %s is equal to %.2f %s\n", amount, baseCurrency, convertedAmount, targetCurrency);
        } else {
            System.out.println("Conversion rate for the specified target currency not found.");
        }

        scanner.close();
    }
}
