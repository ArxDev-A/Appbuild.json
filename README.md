import com.vaadin.flow.component.button.Button;
import com.vaadin.flow.component.html.H1;
import com.vaadin.flow.component.html.Paragraph;
import com.vaadin.flow.component.html.Span;
import com.vaadin.flow.component.icon.Icon;
import com.vaadin.flow.component.icon.VaadinIcon;
import com.vaadin.flow.component.notification.Notification;
import com.vaadin.flow.component.orderedlayout.HorizontalLayout;
import com.vaadin.flow.component.orderedlayout.VerticalLayout;
import com.vaadin.flow.component.radiobutton.RadioButtonGroup;
import com.vaadin.flow.component.textfield.TextField;
import com.vaadin.flow.component.upload.Upload;
import com.vaadin.flow.component.upload.receivers.MemoryBuffer;
import com.vaadin.flow.router.PageTitle;
import com.vaadin.flow.router.Route;
import com.vaadin.flow.server.StreamResource;
import com.vaadin.flow.theme.lumo.Lumo;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;

@Route("")
@PageTitle("DexBuilder - Web & ZIP to APK Compiler")
public class DexBuilderView extends VerticalLayout {

    private final MemoryBuffer iconBuffer = new MemoryBuffer();
    private final MemoryBuffer zipBuffer = new MemoryBuffer();

    public DexBuilderView() {
        // Set tema gelap (Dark Theme) & tata letak utama
        getThemeList().add(Lumo.DARK);
        setSizeFull();
        setAlignItems(Alignment.CENTER);
        setJustifyContentMode(JustifyContentMode.CENTER);

        // Background Gradient Estetik langsung lewat Java Style
        getElement().getStyle()
                .set("background", "radial-gradient(circle at 50% 50%, #1e1b4b, #0f172a, #020617)")
                .set("font-family", "'Segoe UI', Roboto, sans-serif");

        // --- HEADER ---
        H1 title = new H1("DexBuilder");
        title.getStyle()
                .set("margin", "0")
                .set("background", "linear-gradient(to right, #818cf8, #c084fc, #f472b6)")
                .set("-webkit-background-clip", "text")
                .set("-webkit-text-fill-color", "transparent")
                .set("font-weight", "900");

        Span developerTag = new Span("Developed by Athar");
        developerTag.getStyle()
                .set("background", "rgba(99, 102, 241, 0.2)")
                .set("color", "#a5b4fc")
                .set("padding", "4px 12px")
                .set("border-radius", "20px")
                .set("font-size", "0.8em")
                .set("border", "1px solid rgba(99, 102, 241, 0.4)");

        HorizontalLayout header = new HorizontalLayout(new Icon(VaadinIcon.CUBES), title, developerTag);
        header.setAlignItems(Alignment.CENTER);

        Paragraph subtitle = new Paragraph("Ubah Website URL atau File ZIP HTML menjadi Aplikasi APK Android.");
        subtitle.getStyle().set("color", "#94a3b8").set("font-size", "0.9em");

        // --- FORM CARD (Glassmorphism Style) ---
        VerticalLayout formCard = new VerticalLayout();
        formCard.setWidth("480px");
        formCard.getStyle()
                .set("background", "rgba(255, 255, 255, 0.05)")
                .set("backdrop-filter", "blur(16px)")
                .set("border-radius", "24px")
                .set("padding", "32px")
                .set("border", "1px solid rgba(255, 255, 255, 0.1)")
                .set("box-shadow", "0 20px 40px rgba(0,0,0,0.5)");

        // 1. Input Nama Aplikasi
        TextField appNameField = new TextField("NAMA APLIKASI (APK)");
        appNameField.setPlaceholder("Contoh: Toko Saya App");
        appNameField.setPrefixComponent(new Icon(VaadinIcon.MOBILE));
        appNameField.setWidthFull();

        // 2. Pilih Sumber (URL / ZIP)
        RadioButtonGroup<String> sourceType = new RadioButtonGroup<>();
        sourceType.setLabel("SUMBER APLIKASI");
        sourceType.setItems("Website URL", "Upload File ZIP");
        sourceType.setValue("Website URL");

        // Input URL
        TextField urlField = new TextField("URL WEBSITE");
        urlField.setPlaceholder("https://websiteanda.com");
        urlField.setPrefixComponent(new Icon(VaadinIcon.GLOBE));
        urlField.setWidthFull();

        // Upload ZIP File
        Upload zipUpload = new Upload(zipBuffer);
        zipUpload.setUploadButton(new Button("Pilih File ZIP Web", new Icon(VaadinIcon.UPLOAD_ALT)));
        zipUpload.setAcceptedFileTypes(".zip");
        zipUpload.setWidthFull();
        zipUpload.setVisible(false);

        // Toggle Tampilan URL vs ZIP
        sourceType.addValueChangeListener(event -> {
            boolean isUrl = "Website URL".equals(event.getValue());
            urlField.setVisible(isUrl);
            zipUpload.setVisible(!isUrl);
        });

        // 3. Upload Logo / Icon APK
        Upload iconUpload = new Upload(iconBuffer);
        iconUpload.setUploadButton(new Button("Upload Logo / Icon APK", new Icon(VaadinIcon.PICTURE)));
        iconUpload.setAcceptedFileTypes("image/png", "image/jpeg");
        iconUpload.setWidthFull();

        // 4. Tombol Build & Download APK
        Button buildButton = new Button("Build & Download APK", new Icon(VaadinIcon.COG));
        buildButton.setWidthFull();
        buildButton.getStyle()
                .set("background", "#4f46e5")
                .set("color", "white")
                .set("font-weight", "bold")
                .set("border-radius", "12px")
                .set("margin-top", "16px")
                .set("cursor", "pointer")
                .set("box-shadow", "0 0 20px rgba(79, 70, 229, 0.4)");

        // Listener saat tombol diklik
        buildButton.addClickListener(e -> {
            if (appNameField.isEmpty()) {
                Notification.show("Silakan masukkan Nama Aplikasi terlebih dahulu!", 3000, Notification.Position.MIDDLE);
                return;
            }

            // Membuat resource stream untuk download APK
            StreamResource resource = new StreamResource(
                    appNameField.getValue().replaceAll("[^a-zA-Z0-9]", "_").toLowerCase() + "_DexBuilder.apk",
                    () -> generateApkStream(appNameField.getValue(), sourceType.getValue(), urlField.getValue())
            );

            // Trigger download otomatis
            buildButton.getUI().ifPresent(ui -> ui.getPage().open(resource));
            Notification.show("Mulai mengunduh APK: " + appNameField.getValue(), 3000, Notification.Position.BOTTOM_CENTER);
        });

        // Menyusun elemen ke dalam Card Form
        formCard.add(appNameField, sourceType, urlField, zipUpload, iconUpload, buildButton);

        // Menambahkan elemen ke tampilan utama
        add(header, subtitle, formCard);
    }

    /**
     * Fungsi Java untuk menyusun struktur file APK dalam bentuk Stream
     */
    private ByteArrayInputStream generateApkStream(String appName, String sourceType, String url) {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        try (ZipOutputStream zos = new ZipOutputStream(baos)) {

            // 1. Tulis Metadata Konfigurasi APK
            String configContent = String.format(
                    "App Name: %s\nDeveloper: Athar\nSource: %s\nURL: %s\nCompiled by DexBuilder Java Engine",
                    appName, sourceType, url
            );
            ZipEntry configEntry = new ZipEntry("assets/dexbuilder_config.txt");
            zos.putNextEntry(configEntry);
            zos.write(configContent.getBytes(StandardCharsets.UTF_8));
            zos.closeEntry();

            // 2. Masukkan Icon jika di-upload
            if (iconBuffer.getFileName() != null && !iconBuffer.getFileName().isEmpty()) {
                ZipEntry iconEntry = new ZipEntry("res/drawable/ic_launcher.png");
                zos.putNextEntry(iconEntry);
                zos.write(iconBuffer.getInputStream().readAllBytes());
                zos.closeEntry();
            }

            // 3. Masukkan Asset Web jika ZIP di-upload
            if (zipBuffer.getFileName() != null && !zipBuffer.getFileName().isEmpty()) {
                ZipEntry zipEntry = new ZipEntry("assets/www.zip");
                zos.putNextEntry(zipEntry);
                zos.write(zipBuffer.getInputStream().readAllBytes());
                zos.closeEntry();
            }

            // Dummy Bytecode untuk menyimulasikan file APK
            ZipEntry dexEntry = new ZipEntry("classes.dex");
            zos.putNextEntry(dexEntry);
            zos.write("DEX_BUILDER_JAVA_BYTECODE_ATHAR".getBytes(StandardCharsets.UTF_8));
            zos.closeEntry();

        } catch (IOException ex) {
            ex.printStackTrace();
        }
        return new ByteArrayInputStream(baos.toByteArray());
    }
}
