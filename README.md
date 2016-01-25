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
  
  ------------------------------
  
  
  
  # EditText TextValidator
  
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

# Singleton example

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

