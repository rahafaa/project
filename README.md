
import android.Manifest;
import android.annotation.SuppressLint;
import android.app.Activity;
import android.app.Dialog;
import android.app.ProgressDialog;
import android.content.Context;
import android.content.pm.PackageManager;
import android.location.Location;
import android.location.LocationListener;
import android.location.LocationManager;
import android.media.MediaPlayer;
import android.net.Uri;
import android.os.Bundle;
import android.os.Handler;
import android.util.Log;
import android.view.View;
import android.view.ViewGroup;
import android.view.Window;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ImageView;
import android.widget.TextView;
import android.widget.Toast;

import com.android.volley.Request;
import com.android.volley.VolleyError;
import com.example.myapplication.network.NetworkController;
import com.squareup.picasso.Picasso;

import java.util.HashMap;

public class MainActivity extends AppCompatActivity 
implements NetworkController.ResultListener,   LocationListener {
    public static String SERVER_IP = "192.168.10.7";
    public static String SERVER_URL = "http://" + SERVER_IP + "/sign/server.php";
    public static String SERVER_IMG = "http://" + SERVER_IP + "/sign/img/";
    LocationManager lm;
    TextView timerTextView;
    long startTime = 0;
    Dialog externalDialog;
    Button b;
    EditText serverIp;
    static int current_index = 0;
    double longitude;
    double latitude;
    Handler timerHandler = new Handler();
    Runnable timerRunnable = new Runnable() {

        @Override
        public void run() {

            lm = (LocationManager) getSystemService(Context.LOCATION_SERVICE);
            if (checkSelfPermission(Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED && checkSelfPermission(Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
 Log.d("PERMISSION", "PERMISSION");
   return;}
if (checkSelfPermission(Manifest.permission.INTERNET) != PackageManager.PERMISSION_GRANTED ) {
  
Toast.makeText(MainActivity.this, "INTERNET ACCESS", Toast.LENGTH_LONG).show();
return;}
lm.requestLocationUpdates(LocationManager.NETWORK_PROVIDER, 0, 0 , (LocationListener) MainActivity.this);

            timerHandler.postDelayed(this, 10000);
        }
    };

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        serverIp = (EditText) findViewById(R.id.serverIP);

        b = (Button) findViewById(R.id.testbtn);
        b.setText("start");
        b.setOnClickListener(new View.OnClickListener() {


            @Override
            public void onClick(View v) {
                SERVER_IP = serverIp.getText().toString().trim();
                SERVER_URL = "http://" + SERVER_IP + "/sign/server.php";
                SERVER_IMG = "http://" + SERVER_IP + "/sign/img/";
                b = (Button) v;
                if (b.getText().equals("stop")) {
                    timerHandler.removeCallbacks(timerRunnable);
                    b.setText("start");
                } else {
                    startTime = System.currentTimeMillis();
                    timerHandler.postDelayed(timerRunnable, 10000);
                    b.setText("stop");
                }
            }
        });
    }


    @Override
    public void onResult(int requestCode, boolean isSuccess, String result, VolleyError volleyError, ProgressDialog progressDialog) {

        if(isSuccess) {
            current_index = 0;
            //show dialog of signs and stop timer
            Toast.makeText(MainActivity.this, result , Toast.LENGTH_LONG).show();
            timerHandler.removeCallbacks(timerRunnable);
            b.setText("start");
            if(result.equals("NO")) {
                return;
            }
            final String[] arr = result.trim().split("#");
            try {
                Uri noti = Uri.parse("android.resource://com.example.myapplication/raw/" + arr[current_index].trim());
                final MediaPlayer mp = MediaPlayer.create(this, noti);

                mp.start();
                mp.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
                    public void onCompletion(MediaPlayer mp) {
                        mp.release();
                    }
                });

            } catch ( Exception e) {
                Uri noti = Uri.parse("android.resource://com.example.myapplication/raw/notification");
                final MediaPlayer mp = MediaPlayer.create(this, noti);

                mp.start();
                mp.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
                    public void onCompletion(MediaPlayer mp) {
                        mp.release();
                    }
                });
            }
            final String[] arr1 =result.trim().split("#");
            externalDialog = new Dialog(MainActivity.this);
            externalDialog.setContentView(R.layout.dialog_layout);
            externalDialog.setTitle("signs around you");
            externalDialog.setCancelable(true);
            int width = (int)(getResources().getDisplayMetrics().widthPixels*0.70);
            int height = (int)(getResources().getDisplayMetrics().heightPixels*0.80);

            final ImageView  signView = externalDialog.findViewById(R.id.signImageView);


            Picasso.with(this).load(SERVER_IMG + arr[current_index] + ".png").into(signView);
            Window window = externalDialog.getWindow();
            window.setLayout( width,height);

            signView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View view) {
                    current_index++;
                    if(current_index < arr.length)  {
                        Picasso.with(MainActivity.this).load(SERVER_IMG + arr[current_index] + ".png").into(signView);
                    } else {
                        externalDialog.dismiss();

                        startTime = System.currentTimeMillis();
                        timerHandler.postDelayed(timerRunnable, 10000);
                        b.setText("stop");
                    }

                }
            });
            externalDialog.show();


        }

        else {

        }
    }


        @Override
    public void onPause() {
        super.onPause();
        timerHandler.removeCallbacks(timerRunnable);
        Button b = (Button)findViewById(R.id.testbtn);
        b.setText("start");
    }


    @Override
    public void onLocationChanged(Location location) {
        lm.removeUpdates(this);
        latitude = location.getLatitude();
        longitude = location.getLongitude();
        Toast.makeText(MainActivity.this, "‘ UPDATED  latitude:" + latitude + " longitude:" + longitude, Toast.LENGTH_SHORT).show();

        HashMap<String, String> params = new HashMap<String, String>();

        params.put("latitude",longitude+"");
        params.put("longitude", latitude+"");
       // Toast.makeText(MainActivity.this, "Parameter sen to server"  + latitude + "//" + longitude, Toast.LENGTH_LONG).show();
        NetworkController.getInstance().connect(MainActivity.this,0, SERVER_URL  , Request.Method.POST, params, MainActivity.this);

    }

    @Override
    public void onStatusChanged(String s, int i, Bundle bundle) {

    }

    @Override
    public void onProviderEnabled(String s) {

    }

    @Override
    public void onProviderDisabled(String s) {

    }

    @Override
    public void onPointerCaptureChanged(boolean hasCapture) {

    }
}


