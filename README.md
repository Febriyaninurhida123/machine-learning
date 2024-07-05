# machine-learning
```
Alifia Ananda Putri (312210168)
Febriyani nurhida (312210222)
```

## Face Recognition
Untuk pengenalan wajah, Anda harus menggunakan gambar dengan dimensi minimal 480x360 piksel. Agar ML Kit dapat mendeteksi wajah secara akurat, gambar input harus berisi wajah yang direpresentasikan oleh data piksel yang memadai. Secara umum, setiap wajah yang ingin Anda deteksi dalam gambar harus berukuran minimal 100x100 piksel. Jika Anda ingin mendeteksi kontur wajah, ML Kit memerlukan input resolusi yang lebih tinggi: setiap wajah harus berukuran minimal 200x200 piksel.

Jika Anda mendeteksi wajah dalam aplikasi real-time, sebaiknya pertimbangkan juga dimensi keseluruhan gambar input. Gambar yang lebih kecil dapat diproses lebih cepat. Jadi, untuk mengurangi latensi, ambil gambar dengan resolusi lebih rendah. Namun, perhatikan persyaratan akurasi di atas dan pastikan wajah subjek menempati gambar sebanyak mungkin. Lihat juga tips untuk meningkatkan performa real-time.

Fokus gambar yang buruk juga dapat memengaruhi akurasi. Jika Anda tidak mendapatkan hasil yang dapat diterima, minta pengguna untuk mengambil ulang gambar.

Orientasi wajah terhadap arah kamera juga dapat memengaruhi fitur wajah yang terdeteksi oleh ML Kit. Baca Konsep Deteksi Wajah.

## Java
```
package com.example.rtku;

import android.Manifest;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.graphics.Bitmap;
import android.graphics.Rect;
import android.os.Bundle;
import android.util.Log;
import android.view.TextureView;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import com.google.android.gms.tasks.OnFailureListener;
import com.google.android.gms.tasks.OnSuccessListener;
import com.google.mlkit.vision.common.InputImage;
import com.google.mlkit.vision.face.Face;
import com.google.mlkit.vision.face.FaceDetection;
import com.google.mlkit.vision.face.FaceDetector;
import com.google.mlkit.vision.face.FaceDetectorOptions;
import com.example.rtku.Helper.CameraHelper;
import com.example.rtku.User.MainActivity;

import java.util.List;

public class FaceDetectionActivity extends AppCompatActivity {

    private static final int REQUEST_CAMERA_PERMISSION = 200;
    private TextureView textureView;
    private CameraHelper cameraHelper;
    private Button buttonCapture;
    private Button buttonSwitchCamera;
    private boolean isUsingFrontCamera = true;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.facedetection);

        textureView = findViewById(R.id.textureView);
        buttonCapture = findViewById(R.id.buttonCapture);
        buttonSwitchCamera = findViewById(R.id.buttonSwitchCamera);

        if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA)
                != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this,
                    new String[]{Manifest.permission.CAMERA},
                    REQUEST_CAMERA_PERMISSION);
        } else {
            startCamera();
        }

        buttonCapture.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Bitmap bitmap = textureView.getBitmap();
                if (bitmap != null) {
                    detectFaces(bitmap);
                }
            }
        });

        buttonSwitchCamera.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                switchCamera();
            }
        });
    }

    private void startCamera() {
        cameraHelper = new CameraHelper(textureView, isUsingFrontCamera);
        cameraHelper.startCamera();
    }

    private void switchCamera() {
        isUsingFrontCamera = !isUsingFrontCamera;
        cameraHelper.stopCamera();
        startCamera();
    }

    private void detectFaces(Bitmap bitmap) {
        InputImage image = InputImage.fromBitmap(bitmap, 0);

        FaceDetectorOptions options = new FaceDetectorOptions.Builder()
                .setPerformanceMode(FaceDetectorOptions.PERFORMANCE_MODE_ACCURATE)
                .setLandmarkMode(FaceDetectorOptions.LANDMARK_MODE_ALL)
                .setClassificationMode(FaceDetectorOptions.CLASSIFICATION_MODE_ALL)
                .build();

        FaceDetector detector = FaceDetection.getClient(options);

        detector.process(image)
                .addOnSuccessListener(new OnSuccessListener<List<Face>>() {
                    @Override
                    public void onSuccess(List<Face> faces) {
                        for (Face face : faces) {
                            Rect bounds = face.getBoundingBox();
                            Log.d("Face Detection", "Face detected with bounding box: " + bounds.toString());
                        }
                        Toast.makeText(FaceDetectionActivity.this, "Face detection completed", Toast.LENGTH_SHORT).show();

                        // Start MainActivity after successful face detection
                        Intent intent = new Intent(FaceDetectionActivity.this, MainActivity.class);
                        startActivity(intent);
                        finish(); // Optional: finish FaceDetectionActivity if not needed anymore
                    }
                })
                .addOnFailureListener(new OnFailureListener() {
                    @Override
                    public void onFailure(@NonNull Exception e) {
                        Log.e("Face Detection", "Failed to detect faces", e);
                        Toast.makeText(FaceDetectionActivity.this, "Failed to detect faces", Toast.LENGTH_SHORT).show();
                    }
                });
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == REQUEST_CAMERA_PERMISSION) {
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                startCamera();
            } else {
                Toast.makeText(this, "Permission denied", Toast.LENGTH_SHORT).show();
            }
        }
    }
}
```

## XML
```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".FaceDetectionActivity">

    <TextureView
        android:id="@+id/textureView"
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:layout_centerInParent="true" />

    <View
        android:id="@+id/overlay"
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:layout_centerInParent="true"
        android:background="@drawable/face_overlay" />

    <Button
        android:id="@+id/buttonCapture"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Switch Camera"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true" />

    <Button
        android:id="@+id/buttonSwitchCamera"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Switch Camera"
        android:layout_alignParentTop="true"
        android:layout_centerHorizontal="true" />

</RelativeLayout>
```
## hasil

![5](https://github.com/Febriyaninurhida123/machine-learning/assets/90132092/c49164eb-aae9-4549-a49f-4a54eca93489)
