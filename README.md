# GoogleAdsImplementation

## Implementaion of Interstitial Ads :-

Seperate class for handling Interstitial ads named GoogleInterstitialAdHandler-

```java
public class GoogleInterstitialAdHandler {

    public static int ADS_VISIBLE_LIMIT = 1;
    public static int current = 1;
    public InterstitialAd mInterstitialAd;
    public AdsCloseListener adsCloseListener;
    private static final String INTERSTITAL_ID = "ca-app-pub-3940256099942544/1033173712"; //test
    public static final String TAG = "googleAds";

    public void initAndLoad(Context context){
        if (Utility.isPremium()) return;
        if (Utility.isRewardedPremium()){
            return;
        }
        if( Utility.isTestLab() ) return;
        if(mInterstitialAd != null && mInterstitialAd.isLoaded())
            return;
       
        mInterstitialAd = new InterstitialAd(context);
        mInterstitialAd.setAdUnitId(INTERSTITAL_ID);
        mInterstitialAd.setAdListener(new AdListener() {
            @Override
            public void onAdClosed() {
                // Load the next interstitial.
                if(adsCloseListener != null){
                    adsCloseListener.onAdClosed();
                }
                mInterstitialAd.loadAd(new AdRequest.Builder().build());
            }

            @Override
            public void onAdLoaded() {
                super.onAdLoaded();
                Log.d(TAG, "onAdLoaded: ");
                if (adsLoadedListener != null) {
                    adsLoadedListener.onAdLoadedOrFailure();
                }
            }

            @Override
            public void onAdFailedToLoad(LoadAdError loadAdError) {
                super.onAdFailedToLoad(loadAdError);
                Log.d(TAG, "onAdFailedToLoad: ");
                if (adsLoadedListener != null) {
                    adsLoadedListener.onAdLoadedOrFailure();
                }
            }
        });
        mInterstitialAd.loadAd(new AdRequest.Builder().build());
    }

    public boolean showAd(){
        if (Utility.isPremium())
            return false;
        if (Utility.isRewardedPremium()){
            return false;
        }
        try {
            if (mInterstitialAd != null && mInterstitialAd.isLoaded()) {
                mInterstitialAd.show();
                return true;
            } else {
                reloadAd();
                return false;
            }
        }catch (Exception e){
            return false;
        }
    }

    private void reloadAd() {
        try{
            // Request a new ad if one isn't already loaded.
            if (mInterstitialAd!= null && !mInterstitialAd.isLoaded()) {
                mInterstitialAd.loadAd(new AdRequest.Builder().build());
            }
        }catch (Exception ignored){

        }
    }

    public boolean showAdWithCounter(){
        if(current >=  ADS_VISIBLE_LIMIT){
            boolean result = showAd();
            if(result)
                current = 1;
            return result;
        } else {
            current ++;
            return false;
        }
    }



    public interface AdsCloseListener {
        void onAdClosed();
    }

    public void setAdLoadedListener(AdsLoadedListener adsLoadedListener) {
        this.adsLoadedListener = adsLoadedListener;
    }

}
```

## Implementaion of Banner Ads :-
### in any layout file add SMART_BANNER 
```xml
<RelativeLayout
             android:id="@+id/ad_header"
             android:layout_width="match_parent"
             android:layout_height="wrap_content"
             android:layout_centerHorizontal="true"
             android:layout_centerInParent="true"
             />
        <com.google.android.gms.ads.AdView
             xmlns:ads="http://schemas.android.com/apk/res-auto"
             android:id="@+id/adView"
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:layout_centerHorizontal="true"
             android:layout_centerInParent="true"
             ads:adSize="SMART_BANNER"
             ads:adUnitId="ca-app-pub-9865115953083848/7757503036">
         </com.google.android.gms.ads.AdView>
</RelativeLayout>
```
### create a seperate class for handling banner ads

```java
public class GoogleBannerAdHandler {

    public Activity mainActivity;
    private AdView bannerAdView;
    private RelativeLayout adHeader;
    private AdRequest adRequest;

    public GoogleBannerAdHandler(Activity mainActivity, AdView linearLayout, RelativeLayout adHeader) {
        if (Utility.isPremium()) {
            bannerAdView = null;
            return;
        }
        if (Utility.isTestLab()) return;
        
        this.mainActivity = mainActivity;
        this.bannerAdView = linearLayout;
        this.adHeader = adHeader;
        adRequest = new AdRequest.Builder().build();
    }

    public void showBannerAd() {

        if (mainActivity == null || bannerAdView == null)
            return;
        if (Utility.isPremium()) {
            bannerAdView = null;
            return;
        }
        if (Utility.isTestLab()) return;

       /* if (BuildConfig.DEBUG)
            return;*/

        if (adHeader == null)return;
        bannerAdView.setAdListener(new AdListener() {
            @Override
            public void onAdLoaded() {
                // Code to be executed when an ad finishes loading.
                bannerAdView.setVisibility(View.VISIBLE);
                adHeader.setVisibility(View.VISIBLE);
            }

            @Override
            public void onAdFailedToLoad(int errorCode) {
                // Code to be executed when an ad request fails.
                bannerAdView.setVisibility(View.GONE);
                adHeader.setVisibility(View.GONE);
            }

            @Override
            public void onAdOpened() {
                // Code to be executed when an ad opens an overlay that
                // covers the screen.
            }

            @Override
            public void onAdLeftApplication() {
                // Code to be executed when the user has left the app.
            }

            @Override
            public void onAdClosed() {
                // Code to be executed when when the user is about to return
                // to the app after tapping on an ad.
            }
        });

        // Request an ad
        bannerAdView.loadAd(adRequest);
    }

    public void onPause() {
        if (bannerAdView != null) {
            bannerAdView.pause();
        }
    }


    public void onResume() {
        if (bannerAdView != null) {
            bannerAdView.resume();
        }
    }


    public void onDestroy() {
        if (bannerAdView != null) {
            bannerAdView.destroy();
        }
    }

}
```
## Implementaion of Native Ads in exit dialog :-

### Create a seperate class for handling Native ads names as GoogleNativeHandlerAd

```java
public class GoogleNativeHandlerAd {
    private UnifiedNativeAd nativeAd;
    public AdLoader.Builder builder;
    
    public GoogleNativeHandlerAd(Context context) {
        if (Utility.isTestLab()) return;
         builder = new AdLoader.Builder(context, "ca-app-pub-3940256099942544/2247696110");//test

        if (nativeAd != null) {
            nativeAd.destroy();
        }

        builder.forUnifiedNativeAd(new UnifiedNativeAd.OnUnifiedNativeAdLoadedListener() {

            // OnUnifiedNativeAdLoadedListener implementation.
            @Override
            public void onUnifiedNativeAdLoaded(UnifiedNativeAd unifiedNativeAd) {
             nativeAd = unifiedNativeAd;
            }

        });
       
        AdLoader adLoader = builder.build();
        adLoader.loadAd(new AdRequest.Builder().build());

    }

    public UnifiedNativeAd getNativeAd(){
        return nativeAd;
    }
}
```

### create a dialog to show native ad

```java
public class GoogleExitDialog {
    public static UnifiedNativeAd nativeAd;

    public static void showNativeAd(final Activity activity, GoogleNativeHandlerAd nativeHandlerAd) {

        try {
            final Dialog main_dialog;
            LayoutInflater dialogLayout = LayoutInflater.from(activity);
            final View DialogView = dialogLayout.inflate(R.layout.activity_ads, null);
            main_dialog = new Dialog(activity, R.style.CustomAlertDialog);
            main_dialog.setContentView(DialogView);
            // You must call destroy on old ads when you are done with them,
            // otherwise you will have a memory leak.
            nativeAd = nativeHandlerAd.getNativeAd();
            FrameLayout frameLayout =
                    DialogView.findViewById(R.id.fl_adplaceholder);
            frameLayout.setVisibility(View.VISIBLE);
            UnifiedNativeAdView adView = (UnifiedNativeAdView) LayoutInflater.from(activity)
                    .inflate(R.layout.ad_unified, null);
            populateUnifiedNativeAdView(nativeAd, adView);
            frameLayout.addView(adView);

            Button exit = DialogView.findViewById(R.id.exit_button);
            exit.setBackgroundColor(Color.parseColor("#FFB300"));
            exit.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    if (main_dialog.isShowing())
                        main_dialog.dismiss();
                    activity.finish();
                }
            });

            Button noButton = DialogView.findViewById(R.id.no_button);
            noButton.setBackgroundColor(Color.parseColor("#FFB300"));
            noButton.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    EventBus.getDefault().postSticky(new EventMsg.RefreshAdGoogle());
                    if (main_dialog.isShowing())
                        main_dialog.dismiss();
                }
            });

            main_dialog.setCancelable(false);
            main_dialog.setCanceledOnTouchOutside(false);
            main_dialog.show();
        } catch (Exception e) {
            activity.finish();
            FirebaseCrashlytics.getInstance().log(e.getMessage());
        }


    }

    public static void populateUnifiedNativeAdView(UnifiedNativeAd nativeAd, UnifiedNativeAdView adView) {
        // Set the media view. Media content will be automatically populated in the media view once
        // adView.setNativeAd() is called.
        MediaView mediaView = adView.findViewById(R.id.ad_media);
        adView.setMediaView(mediaView);

        // --------- populate all the views
    }

}
```

// layout file for exit dialog named as activity_ads

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    xmlns:android="http://schemas.android.com/apk/res/android">

    <FrameLayout
        android:id="@+id/fl_adplaceholder"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#FFB300"
        android:visibility="gone"
        android:padding="5dp"/>

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="5dp">
        <TextView
            android:id="@+id/exit"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textSize="15sp"
            android:layout_margin="10dp"
            android:gravity="center"
            android:textColor="@color/black"
            android:text="@string/are_you_sure_you_want_quit_this_app"/>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_below="@+id/exit"
            android:weightSum="2"
            android:gravity="center"
            android:orientation="horizontal">
            <Button
                android:id="@+id/no_button"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight=".75"
                android:padding="10dp"
                android:layout_marginRight="3dp"
                android:gravity="center"
                android:textColor="@color/white"
                android:text="@string/no"/>
            <Button
                android:id="@+id/exit_button"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight=".75"
                android:padding="10dp"
                android:layout_marginLeft="3dp"
                android:gravity="center"
                android:textColor="@color/white"
                android:text="@string/yes_quit_now"/>
        </LinearLayout>


    </RelativeLayout>
</LinearLayout>
```


### and in any activity like MainActivity or launcher activity we use this classes like this:

```java
public class MainActivity extends AppCompatActivity {
  public GoogleInterstitialAdHandler intertitialAdHandler;
  public GoogleBannerAdHandler bannerAdHandler;
  public GoogleNativeHandlerAd googleNativeHandlerAd;
  
  protected void onCreate(Bundle savedInstanceState) {
  
   intertitialAdHandler = new GoogleInterstitialAdHandler();
   intertitialAdHandler.initAndLoad(AppConfig.getInstance());
   
   RelativeLayout adHeader = findViewById(R.id.ad_header);
   AdView mAdView = findViewById(R.id.adView);
   bannerAdHandler = new GoogleBannerAdHandler(this, mAdView, adHeader);
   bannerAdHandler.showBannerAd();
   
  }
  
  protected void onResume() {
        bannerAdHandler.onResume();
        super.onResume();
  }
  
  protected void onPause() {
      super.onPause();
      bannerAdHandler.onPause();
  }
  
  protected void onDestroy() {
     bannerAdHandler.onDestroy();
     super.onDestroy();
  }
    
  // show GoogleExitDialog on backPressed  
  public void onBackPressed() {
    showNativeAd();
  }
  
  public void showNativeAd() {
        if (Utility.isPremium()) {
            exit();
        } else {
            if (googleNativeHandlerAd != null && googleNativeHandlerAd.getNativeAd() != null) {
                GoogleExitDialog.showNativeAd(this, googleNativeHandlerAd);
            } else {
                exit();
            }

        }
    }
  
  private void exit() {
   finish(); 
  }
  
  // listener for navigation drawer items and show ad in every item click
  private class NavigationDrawerItemsListener implements AdapterView.OnItemClickListener {

        @Override
        public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
            // when click on any item in navigation drawer first show interstitial ad
            if (intertitialAdHandler != null && intertitialAdHandler.showAdWithCounter()) {
                intertitialAdHandler.setAdsCloseListener(() -> navigateItemClick(position));
            } else {
                navigateItemClick(position);
            }

            drawerLayout.closeDrawer(GravityCompat.START); // close the drawer
        }
    }
  
  private void navigateItemClick(int position) {
        switch (position) {
            case 0:
                materialDrawer.showVideo();
                break;
            case 1:
                showAllVideosFragment();
                break;
       }
    }
}
```

