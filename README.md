# machine-learning
```
Alifia Ananda Putri (312210168)
Febriyani nurhida (312210222)
```

## 
Cobalah
Coba utak-atik aplikasi contoh ini untuk melihat contoh penggunaan API ini.
Coba kode sendiri dengan codelab.
Sebelum memulai
API ini memerlukan Android API level 21 atau yang lebih tinggi. Pastikan file build aplikasi menggunakan nilai minSdkVersion 21 atau lebih tinggi.
Dalam file build.gradle level project, pastikan Anda memasukkan repositori Maven Google di bagian buildscript dan allprojects.

Tambahkan dependensi untuk library Android ML Kit ke file gradle level aplikasi modul Anda, biasanya app/build.gradle. Pilih salah satu dependensi berikut berdasarkan kebutuhan Anda:

Untuk memaketkan model dengan aplikasi Anda:


dependencies {
  // ...
  // Use this dependency to bundle the model with your app
  implementation 'com.google.mlkit:face-detection:16.1.6'
}
Untuk menggunakan model di Layanan Google Play:


dependencies {
  // ...
  // Use this dependency to use the dynamically downloaded model in Google Play Services
  implementation 'com.google.android.gms:play-services-mlkit-face-detection:17.1.0'
}
Jika memilih untuk menggunakan model di Layanan Google Play, Anda dapat mengonfigurasi aplikasi untuk otomatis mendownload model ke perangkat setelah aplikasi diinstal dari Play Store. Untuk melakukannya, tambahkan deklarasi berikut ke file AndroidManifest.xml aplikasi Anda:


<application ...>
      ...
      <meta-data
          android:name="com.google.mlkit.vision.DEPENDENCIES"
          android:value="face" >
      <!-- To use multiple models: android:value="face,model2,model3" -->
</application>
Anda juga dapat memeriksa ketersediaan model secara eksplisit dan meminta download melalui layanan Google Play ModuleInstallClient API.

Jika Anda tidak mengaktifkan download model waktu penginstalan atau meminta download eksplisit, model akan didownload saat pertama kali Anda menjalankan detektor. Permintaan yang Anda buat sebelum download selesai tidak memberikan hasil apa pun.

Panduan gambar input
Untuk pengenalan wajah, Anda harus menggunakan gambar dengan dimensi minimal 480x360 piksel. Agar ML Kit dapat mendeteksi wajah secara akurat, gambar input harus berisi wajah yang direpresentasikan oleh data piksel yang memadai. Secara umum, setiap wajah yang ingin Anda deteksi dalam gambar harus berukuran minimal 100x100 piksel. Jika Anda ingin mendeteksi kontur wajah, ML Kit memerlukan input resolusi yang lebih tinggi: setiap wajah harus berukuran minimal 200x200 piksel.

Jika Anda mendeteksi wajah dalam aplikasi real-time, sebaiknya pertimbangkan juga dimensi keseluruhan gambar input. Gambar yang lebih kecil dapat diproses lebih cepat. Jadi, untuk mengurangi latensi, ambil gambar dengan resolusi lebih rendah. Namun, perhatikan persyaratan akurasi di atas dan pastikan wajah subjek menempati gambar sebanyak mungkin. Lihat juga tips untuk meningkatkan performa real-time.

Fokus gambar yang buruk juga dapat memengaruhi akurasi. Jika Anda tidak mendapatkan hasil yang dapat diterima, minta pengguna untuk mengambil ulang gambar.

Orientasi wajah terhadap arah kamera juga dapat memengaruhi fitur wajah yang terdeteksi oleh ML Kit. Baca Konsep Deteksi Wajah.

1. Mengonfigurasi detektor wajah
Jika ingin mengubah salah satu setelan default pada detektor wajah sebelum menerapkan deteksi wajah ke gambar, tentukan setelan tersebut dengan objek FaceDetectorOptions. Anda dapat mengubah setelan berikut:
Setelan
setPerformanceMode	PERFORMANCE_MODE_FAST (default) | PERFORMANCE_MODE_ACCURATE
Mendukung kecepatan atau akurasi saat mendeteksi wajah.

setLandmarkMode	LANDMARK_MODE_NONE (default) | LANDMARK_MODE_ALL
Mencoba mengidentifikasi "landmark" wajah: mata, telinga, hidung, pipi, mulut, dll.

setContourMode	CONTOUR_MODE_NONE (default) | CONTOUR_MODE_ALL
Apakah mendeteksi kontur fitur wajah. Kontur dideteksi hanya untuk wajah yang paling terlihat dalam gambar.

setClassificationMode	CLASSIFICATION_MODE_NONE (default) | CLASSIFICATION_MODE_ALL
Mengklasifikasikan wajah ke dalam beberapa kategori atau tidak, seperti "tersenyum", dan "mata terbuka".

setMinFaceSize	float (default: 0.1f)
Menetapkan ukuran wajah terkecil yang diinginkan, yang dinyatakan sebagai rasio lebar kepala terhadap lebar gambar.

enableTracking	false (default) | true
Menetapkan ID pada wajah atau tidak, yang dapat digunakan untuk melacak wajah di seluruh gambar.

Perhatikan bahwa saat deteksi kontur diaktifkan, hanya satu wajah yang terdeteksi, sehingga pelacakan wajah tidak memberikan hasil yang berguna. Karena alasan ini, dan untuk meningkatkan kecepatan deteksi, jangan aktifkan deteksi kontur dan pelacakan wajah.

Contoh:

Kotlin
Java

// High-accuracy landmark detection and face classification
val highAccuracyOpts = FaceDetectorOptions.Builder()
        .setPerformanceMode(FaceDetectorOptions.PERFORMANCE_MODE_ACCURATE)
        .setLandmarkMode(FaceDetectorOptions.LANDMARK_MODE_ALL)
        .setClassificationMode(FaceDetectorOptions.CLASSIFICATION_MODE_ALL)
        .build()

// Real-time contour detection
val realTimeOpts = FaceDetectorOptions.Builder()
        .setContourMode(FaceDetectorOptions.CONTOUR_MODE_ALL)
        .build()
2. Menyiapkan gambar input
Untuk mendeteksi wajah dalam gambar, buat objek InputImage dari Bitmap, media.Image, ByteBuffer, array byte, atau file di perangkat. Lalu, teruskan objek InputImage ke metode process FaceDetector.
Untuk deteksi wajah, Anda harus menggunakan gambar dengan dimensi yang berukuran minimal 480x360 piksel. Jika Anda mendeteksi wajah secara real time, pengambilan frame pada resolusi minimum ini dapat membantu mengurangi latensi.

Anda dapat membuat objek InputImage dari berbagai sumber, yang masing-masing langkahnya dijelaskan di bawah.

Menggunakan media.Image
Untuk membuat objek InputImage dari objek media.Image, seperti saat mengambil gambar dari kamera perangkat, teruskan objek media.Image dan rotasi gambar ke InputImage.fromMediaImage().

Jika Anda menggunakan library CameraX, class OnImageCapturedListener dan ImageAnalysis.Analyzer menghitung nilai rotasi untuk Anda.

Kotlin
Java

private class YourImageAnalyzer : ImageAnalysis.Analyzer {

    override fun analyze(imageProxy: ImageProxy) {
        val mediaImage = imageProxy.image
        if (mediaImage != null) {
            val image = InputImage.fromMediaImage(mediaImage, imageProxy.imageInfo.rotationDegrees)
            // Pass image to an ML Kit Vision API
            // ...
        }
    }
}
Jika tidak menggunakan library kamera yang memberi derajat rotasi gambar, Anda dapat menghitungnya dari derajat rotasi perangkat dan orientasi sensor kamera pada perangkat:

Kotlin
Java

private val ORIENTATIONS = SparseIntArray()

init {
    ORIENTATIONS.append(Surface.ROTATION_0, 0)
    ORIENTATIONS.append(Surface.ROTATION_90, 90)
    ORIENTATIONS.append(Surface.ROTATION_180, 180)
    ORIENTATIONS.append(Surface.ROTATION_270, 270)
}

/**
 * Get the angle by which an image must be rotated given the device's current
 * orientation.
 */
@RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
@Throws(CameraAccessException::class)
private fun getRotationCompensation(cameraId: String, activity: Activity, isFrontFacing: Boolean): Int {
    // Get the device's current rotation relative to its "native" orientation.
    // Then, from the ORIENTATIONS table, look up the angle the image must be
    // rotated to compensate for the device's rotation.
    val deviceRotation = activity.windowManager.defaultDisplay.rotation
    var rotationCompensation = ORIENTATIONS.get(deviceRotation)

    // Get the device's sensor orientation.
    val cameraManager = activity.getSystemService(CAMERA_SERVICE) as CameraManager
    val sensorOrientation = cameraManager
            .getCameraCharacteristics(cameraId)
            .get(CameraCharacteristics.SENSOR_ORIENTATION)!!

    if (isFrontFacing) {
        rotationCompensation = (sensorOrientation + rotationCompensation) % 360
    } else { // back-facing
        rotationCompensation = (sensorOrientation - rotationCompensation + 360) % 360
    }
    return rotationCompensation
}
Kemudian, teruskan objek media.Image dan nilai derajat rotasi ke InputImage.fromMediaImage():

Kotlin
Java

val image = InputImage.fromMediaImage(mediaImage, rotation)

Menggunakan URI file
Untuk membuat objek InputImage dari URI file, teruskan konteks aplikasi dan URI file ke InputImage.fromFilePath(). Hal ini berguna saat Anda menggunakan intent ACTION_GET_CONTENT untuk meminta pengguna memilih gambar dari aplikasi galeri mereka.

Kotlin
Java

val image: InputImage
try {
    image = InputImage.fromFilePath(context, uri)
} catch (e: IOException) {
    e.printStackTrace()
}

Menggunakan ByteBuffer atau ByteArray
Untuk membuat objek InputImage dari ByteBuffer atau ByteArray, pertama-tama hitung derajat rotasi gambar seperti yang dijelaskan sebelumnya untuk input media.Image. Kemudian, buat objek InputImage dengan buffer atau array, beserta tinggi, lebar, format encoding warna, dan derajat rotasi gambar:

Kotlin
Java

val image = InputImage.fromByteBuffer(
        byteBuffer,
        /* image width */ 480,
        /* image height */ 360,
        rotationDegrees,
        InputImage.IMAGE_FORMAT_NV21 // or IMAGE_FORMAT_YV12
)

// Or:
val image = InputImage.fromByteArray(
        byteArray,
        /* image width */ 480,
        /* image height */ 360,
        rotationDegrees,
        InputImage.IMAGE_FORMAT_NV21 // or IMAGE_FORMAT_YV12
)
Menggunakan Bitmap
Untuk membuat objek InputImage dari objek Bitmap, buat deklarasi berikut:

Kotlin
Java

val image = InputImage.fromBitmap(bitmap, 0)

Gambar direpresentasikan oleh objek Bitmap bersama dengan derajat rotasi.

3. Mendapatkan instance FaceDetector
Kotlin
Java

val detector = FaceDetection.getClient(options)
// Or, to use the default option:
// val detector = FaceDetection.getClient();
4. Memproses gambar
Teruskan gambar ke metode process:
Kotlin
Java

val result = detector.process(image)
        .addOnSuccessListener { faces ->
            // Task completed successfully
            // ...
        }
        .addOnFailureListener { e ->
            // Task failed with an exception
            // ...
        }
Catatan: Jika Anda menggunakan CameraX API, pastikan untuk menutup ImageProxy setelah selesai menggunakannya, misalnya, dengan menambahkan OnCompleteListener ke Task yang ditampilkan dari metode process. Lihat class VisionProcessorBase di aplikasi contoh panduan memulai untuk mengetahui contohnya.
5. Mendapatkan informasi tentang wajah yang terdeteksi
Jika operasi deteksi wajah berhasil, daftar objek Face akan diteruskan ke pemroses peristiwa sukses. Setiap objek Face mewakili wajah yang terdeteksi dalam gambar. Untuk setiap wajah, Anda bisa mendapatkan koordinat pembatasnya di gambar input, serta informasi lain yang dapat ditemukan oleh detektor wajah sesuai dengan konfigurasi yang Anda tetapkan. Contoh:
Kotlin
Java

for (face in faces) {
    val bounds = face.boundingBox
    val rotY = face.headEulerAngleY // Head is rotated to the right rotY degrees
    val rotZ = face.headEulerAngleZ // Head is tilted sideways rotZ degrees

    // If landmark detection was enabled (mouth, ears, eyes, cheeks, and
    // nose available):
    val leftEar = face.getLandmark(FaceLandmark.LEFT_EAR)
    leftEar?.let {
        val leftEarPos = leftEar.position
    }

    // If contour detection was enabled:
    val leftEyeContour = face.getContour(FaceContour.LEFT_EYE)?.points
    val upperLipBottomContour = face.getContour(FaceContour.UPPER_LIP_BOTTOM)?.points

    // If classification was enabled:
    if (face.smilingProbability != null) {
        val smileProb = face.smilingProbability
    }
    if (face.rightEyeOpenProbability != null) {
        val rightEyeOpenProb = face.rightEyeOpenProbability
    }

    // If face tracking was enabled:
    if (face.trackingId != null) {
        val id = face.trackingId
    }
}
Contoh kontur wajah
Saat mengaktifkan deteksi kontur wajah, Anda akan melihat daftar titik untuk setiap fitur wajah yang terdeteksi. Titik-titik ini mewakili bentuk fitur. Baca Konsep Deteksi Wajah untuk mengetahui detail tentang cara kontur direpresentasikan.

Gambar berikut mengilustrasikan bagaimana titik-titik ini dipetakan ke wajah, klik gambar untuk memperbesarnya:
