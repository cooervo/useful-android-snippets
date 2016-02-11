# Index

1. [Validator](#validator): Helper class to help verify if strings are name, last names, phone number and email are valid

2. [EditText TextValidator](#textvalidator): An abstract class with abstract method `validate()` to let subclass to customize its implementation

3. [Singleton](#singleton): Singleton is encouraged by Android documentation instead of using Application class and it is great use for models in MVP pattern. This singleton pattern is based on Head First Java Singletton example.

4. [PriceFormatter](#priceformatter): Helper class that takes a number and formats it into a suitable string representation of price

5. [InternalStorage](#internalstorage): Save custom objects in InternalStorage.

6. [EncoderDecoder](#encoderdecoder): A class that uses base 64 to encode and decode

7. [Picasso](#picasso): Picasso example snippet to avoid memory leak

8. [Keyboard-open-close-listener](#keyboard-open-closed-listener): A hack using rootView height to determine when keyboard is opened.

9. [LeakMemoryFixes](#leakmemoryfixes): Common fixes for memory leaks in android


-----------------------------------------------------

# Validator

Helper class to help verify if strings are name, last names, phone number and email are valid

    public class Validator {
   
    /**
     * returns true if param is email
     *
     * @param email
     * @return
     */
    public boolean isEmailValid(CharSequence email) {
        return android.util.Patterns.EMAIL_ADDRESS.matcher(email).matches();
    }


    /**
     * Returns true if string only contains alphabetical chars and whitespaces, ideal for
     * validating names and last names
     *
     * @param name
     * @return
     */
    public boolean isAlpha(String name) {
        return name.matches("[a-zA-Z|\\s]+");
    }

    /**
     * If the string contains at least 7 digits, ideal for validating phone numbers
     *
     * @param phone
     * @return
     */
    public boolean isPhoneNumber(String phone) {
        int count = 0;

        for (int i = 0; i < phone.length(); i++) {

            char c = phone.charAt(i);
            if (Character.isDigit(c)) {
                count++;
            }
        }

        if (count >= 7) {
            return true;
        } else {
            return false;
        }
    }
  }

--------------------------------

# TextValidator

  An abstract class with abstract method `validate()` to let subclass to customize its implementation
  
  
  Usage example:
  
    editTextExample.addTextChangedListener(new TextValidator(emailInput) {
            @Override
            public void validate(TextView tV, String text) {

                if (isValid(text)) {
                    //doSomething();
                } else {
                    //invalidateForm();
                }
            }
        });
  
 
 Class code: 
 
     public abstract class TextValidator implements TextWatcher {
       
         private final TextView textView; //Remember EditText is a TextView so this works for EditText also
    
         public TextValidator(TextView tV){
            textView = tV;
         }
    
         /**
         * Notice abstract method so we make a contract that every instance of TextValidator must implement
         * it's own version of the validate() method, since each editText needs its own validation.
         *
         * @param tV
         * @param text
         */
         public abstract void validate(TextView tV, String text);
    
         @Override
         public void beforeTextChanged(CharSequence s, int start, int count, int after) {
    
         }
    
         @Override
         public void onTextChanged(CharSequence s, int start, int before, int count) {
     
         }
    
         @Override
         public void afterTextChanged(Editable s) {
            String text = textView.getText().toString();
            validate(textView, text); //Notice this is the part that applies validate afterTextChanged()
         }
     }

--------------------------------

# Singleton

Singleton is encouraged by Android documentation instead of using Application class and it is great use for models in MVP pattern. This singleton pattern is based on Head First Java Singletton example.

    public class SharedData {
    
        //Volatile keyword ensures that multiple threads handle the unique/instance correctly
        private volatile static SharedData uniqueInstance;
    
        // ~~~~~~~~~~~~~~~ Configuration of device ~~~~~~~~~~~~~~~
        private int screenWidth;
        private int screenHeight;
        
        // ~~~~~~~~~~~~~~~ Fonts ~~~~~~~~~~~~~~~
        private Typeface main;
        private Typeface secondary;
        
        //Private constructor as we don't need any more instances of this singleton
        private SharedData() {
        }
    
        /**
         * Creates once and only once an instance of this class and returns this instance
         *
         * @return
         */
        public static SharedData getInstance() {
        //Check for an instance and if there isn't one enter the synchronized method
            if (uniqueInstance == null) {
                synchronized (SharedData.class) {
                    //Once in the block, check again and if still null, create the instance
                    if (uniqueInstance == null) {
                        uniqueInstance = new SharedData();
                    }
                }
            }
    
            return uniqueInstance;
        }
        
        public Typeface getMainFont() {
            return main;
        }
    
        public void setMainFont(Context context) {
            main = Typeface.createFromAsset(context.getAssets(), "fonts/VarelaRound-Regular.ttf");
        }
    
        public Typeface getSecondaryFont() {
            return secondary;
        }
    
        public void setSecondaryFont(Context context) {
            secondary = Typeface.createFromAsset(context.getAssets(), "fonts/Abel-Regular.ttf");
        }
    
        public int getScreenWidth() {
            return screenWidth;
        }
    
        public void setScreenWidth(int screenWidth) {
            this.screenWidth = screenWidth;
        }
    
        public int getScreenHeight() {
            return screenHeight;
        }
    
        public void setScreenHeight(int screenHeight) {
            this.screenHeight = screenHeight;
        }
   }

---------
# PriceFormatter

Helper class that takes a number and formats it into a suitable string representation of price
 
* for example: 1000 into 1,000
* for example: 1000.993232 into 1,000.99
* for example: 0.23213132 into 0.23
* for example: 10.000123 into 10
 
      public class PriceFormatter {
    
        //Create an instance of DecimalFormat
        DecimalFormat formatter = (DecimalFormat) NumberFormat.getInstance(Locale.US);
    
        //Now we need instance of DecimalFormatSymbols to use , to separate thousands i.e: 1,000
        DecimalFormatSymbols symbols = formatter.getDecimalFormatSymbols();
    
        public PriceFormatter() {
            //Sets the maximum number of digits after the decimal point
            formatter.setMaximumFractionDigits(2);
            //Sets the minimum number of digits after the decimal point
            formatter.setMinimumFractionDigits(0);
    
            //Set char ',' as the grouping separator
            symbols.setGroupingSeparator(',');
            //Now set for the DecimalFormatter the thousands separator
            formatter.setDecimalFormatSymbols(symbols);
        }
    
        public String formatPrice(BigDecimal decimal){
            return formatter.format(decimal);
        }
      }


------------

# InternalStorage

Save custom objects in InternalStorage:
 
* To write:
     CustomObject myObject = new CustomObject();
     InternalStorage.writeObject(context, "keyExample", myObject);
 
 *To read:
     CustomObject myReadObject = (CustomObject) InternalStorage.readObject(context, "keyExample");

     public final class InternalStorage {

        private InternalStorage() { }
    
        /**
         * Pass a context/activity/service and String key to identify the object you are saving
         *
         * @param context
         * @param key
         * @param object
         */
        public static void writeObject(Context context, String key, Object object) {
    
            try {
    
                FileOutputStream fos = context.openFileOutput(key, Context.MODE_PRIVATE);
                ObjectOutputStream oos = new ObjectOutputStream(fos);
                oos.writeObject(object);
                oos.close();
                fos.close();
    
            } catch (IOException e) {
                e.printStackTrace();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    
        /**
         * Use context/activity/service and a key to retrieve the object
         *
         * @param context
         * @param key
         * @return
         */
        public static Object readObject(Context context, String key) {
    
            Object object = null;
    
            try {
    
                FileInputStream fis = context.openFileInput(key);
                ObjectInputStream ois = new ObjectInputStream(fis);
                object = ois.readObject();
    
            } catch (IOException e) {
                e.printStackTrace();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
    
                return object;
            }
         }
    }
  
 
-----------------------------------------

# EncoderDecoder

A class that uses base 64 to encode and decode strings

    public class EncoderDecoder {
    
        public EncoderDecoder(){
        }
    
        public String encodeString(String s) {
            byte[] data = new byte[0];
    
            try {
                data = s.getBytes("UTF-8");
    
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            } finally {
                String base64Encoded = Base64.encodeToString(data, Base64.DEFAULT);
    
                return base64Encoded;
    
            }
        }
    
        public String decodeString(String encoded) {
            byte[] dataDec = Base64.decode(encoded, Base64.DEFAULT);
            String decodedString = "";
            try {
    
                decodedString = new String(dataDec, "UTF-8");
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
    
            } finally {
    
                return decodedString;
            }
         }
    }
    
    
     
-----------------------------------------

# Picasso

     Picasso.with(ActivityExample.this)                   //Activity context
                    .load(object.getImageUrl())           
                    .placeholder(R.mipmap.placeholder)    //Place holder mipmap if needed
                    .fit()                                //Fits image and avoids memory leak
                    .centerCrop()                         //makes image to not stretch
                    .into(imageView);


-----------------------------------------

# Keyboard-Open-Closed-Listener

Since there is no way to know when software keyboard is opened in android we use the rootView and listen to changes in its height to determine when is keyboard shown.  This is also useful for tablets since there is not change when keyboard is present since rootView height does not changes.


@Override
    protected void onResume() {
        super.onResume();


        mapWrapperKeyboardListener();
    }


    private void mapWrapperKeyboardListener() {
        rootView.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {

            /**
             * Use onGlobalLayout so that once global layout is ready we can getHeight, otherwise heights/widths will
             * give = 0.
             */
            @Override
            public void onGlobalLayout() {

                //Get layouts of mapWrapper
                ViewGroup.LayoutParams mapWrapperParams = mapWrapper.getLayoutParams();

                //Map Height is going to be = rootView.getHeight * 0.4
                int originalMapHeight = (int) Math.round(rootView.getHeight() * 0.4);

                //HeightDiff = rootView.getRootView (the rootView of the rootview duh!) - rootView.getView
                int heightDiff = rootView.getRootView().getHeight() - rootView.getHeight();

                //if keyboard was opened
                if (heightDiff > 100) {
                    
                    //Change mapWrapperParams height to a fraction of originalMapHeight
                    mapWrapperParams.height = (int) Math.round(originalMapHeight * 0.9);
                }

                //if keyboard was closed
                if (heightDiff < 100) {
                    //Change mapWrapperParams height to originalMapHeight
                    mapWrapperParams.height = originalMapHeight;

                }

                //Now set the changed in mapWrapperParams
                mapWrapper.setLayoutParams(mapWrapperParams);

            }
        });
    }


-----------------------------------------

# LeakMemoryFixes

### 1. Fix your contexts: 
Try using the appropiate context: For example since a Toast can be seen in many activities instead of one use `getApplicationContext()` for toasts, and since services can keep running even though an activity has ended start a service with: 

`Intent myService = new Intent(getApplicationContext(), MyService.class)`

Use this table as a quick guide for what context is appropiate:
[![enter image description here][1]][1]

Original [article on context here][2].

### 2. Check that you're actually finishing your services.

For example I have an intentService that use google location service api. And I forgot to call `googleApiClient.disconnect();`:

    //Disconnect from API onDestroy()
        if (googleApiClient.isConnected()) {
            LocationServices.FusedLocationApi.removeLocationUpdates(googleApiClient, GoogleLocationService.this);
            googleApiClient.disconnect();
        }


### 3. Check image and bitmaps usage:
If you are using square's **library Picasso** I found I was leaking memory by not using the `.fit()`, that drastically reduced my memory footprint from 50MB in average to less than 19MB:

    Picasso.with(ActivityExample.this)                   //Activity context
                    .load(object.getImageUrl())           
                    .fit()                                //This avoided the OutOfMemoryError
                    .centerCrop()                         //makes image to not stretch
                    .into(imageView);


### 4. If you are using using broadcast receivers unregister them.

### 5. If you are using `java.util.Observer` (Observer pattern):
Make sure to    to use `deleteObserver(observer);`
 
  [1]: http://i.stack.imgur.com/1o5MI.png
  [2]: https://possiblemobile.com/2013/06/context/
