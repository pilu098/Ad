AndroidManifestfile.xml

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.medicalbill"
    xmlns:tools="http://schemas.android.com/tools">

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Medicalbill"
        tools:targetApi="31">

        <!-- MainActivity: Entry point of the app -->
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:label="Medical Shop Bill">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <!-- BillActivity: Displays the generated bill -->
        <activity
            android:name=".BillActivity"
            android:exported="false"
            android:label="Generated Bill" />

    </application>
</manifest>

==================================================================================================================================================================

BillActivity.java

package com.example.medicalbill;

import android.os.Bundle;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;

public class BillActivity extends AppCompatActivity {

    private TextView txtBill;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_bill);

        txtBill = findViewById(R.id.txt_bill);

        // Get data from Intent
        String customerName = getIntent().getStringExtra("customerName");
        String address = getIntent().getStringExtra("address");
        String city = getIntent().getStringExtra("city");
        String contact = getIntent().getStringExtra("contact");
        String medicine = getIntent().getStringExtra("medicine");
        int quantity = getIntent().getIntExtra("quantity", 0);
        double price = getIntent().getDoubleExtra("price", 0);
        double totalAmount = getIntent().getDoubleExtra("totalAmount", 0);
        double gst = getIntent().getDoubleExtra("gst", 0);
        double finalAmount = getIntent().getDoubleExtra("finalAmount", 0);

        // Display Bill
        String billText = "Medical Shop Bill\n"
                + "-------------------------\n"
                + "Customer: " + customerName + "\n"
                + "Address: " + address + ", " + city + "\n"
                + "Contact: " + contact + "\n"
                + "Medicine: " + medicine + "\n"
                + "Quantity: " + quantity + "\n"
                + "Price per Unit: ₹" + price + "\n"
                + "Total Amount: ₹" + totalAmount + "\n"
                + "GST (18%): ₹" + gst + "\n"
                + "Final Amount: ₹" + finalAmount + "\n"
                + "-------------------------";

        txtBill.setText(billText);
    }
}

=================================================================================================================================================================

MainActivity.java

package com.example.medicalbill;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.ArrayAdapter;
import android.widget.AutoCompleteTextView;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Spinner;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    private EditText edtCustomerName, edtAddress, edtContact, edtQuantity, edtPrice;
    private AutoCompleteTextView autoCity;
    private Spinner spinnerMedicine;
    private Button btnGenerateBill;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Initialize UI elements
        edtCustomerName = findViewById(R.id.edt_customer_name);
        edtAddress = findViewById(R.id.edt_address);
        edtContact = findViewById(R.id.edt_contact);
        edtQuantity = findViewById(R.id.edt_quantity);
        edtPrice = findViewById(R.id.edt_price);
        autoCity = findViewById(R.id.auto_city);
        spinnerMedicine = findViewById(R.id.spinner_medicine);
        btnGenerateBill = findViewById(R.id.btn_generate_bill);

        // Setup City AutoCompleteTextView
        String[] cities = {"Surat", "Ahmedabad", "Vadodara", "Rajkot", "Mumbai", "Delhi"};
        ArrayAdapter<String> cityAdapter = new ArrayAdapter<>(this, android.R.layout.simple_dropdown_item_1line, cities);
        autoCity.setAdapter(cityAdapter);

        // Setup Medicine Dropdown
        String[] medicines = {"Azithromycin", "Amoxicillin", "Lisinopril", "Generic Norvasc", "Generic Synthroid", "Hydrocodone"};
        ArrayAdapter<String> medicineAdapter = new ArrayAdapter<>(this, android.R.layout.simple_spinner_dropdown_item, medicines);
        spinnerMedicine.setAdapter(medicineAdapter);

        // Generate Bill Button Click Event
        btnGenerateBill.setOnClickListener(v -> generateBill());
    }

    private void generateBill() {
        String customerName = edtCustomerName.getText().toString().trim();
        String address = edtAddress.getText().toString().trim();
        String city = autoCity.getText().toString().trim();
        String contact = edtContact.getText().toString().trim();
        String medicine = spinnerMedicine.getSelectedItem().toString();
        String quantityStr = edtQuantity.getText().toString().trim();
        String priceStr = edtPrice.getText().toString().trim();

        // Validation
        if (customerName.isEmpty() || address.isEmpty() || city.isEmpty() || contact.isEmpty() || quantityStr.isEmpty() || priceStr.isEmpty()) {
            Toast.makeText(this, "Please fill all details", Toast.LENGTH_SHORT).show();
            return;
        }
        if (!contact.matches("\\d{10}")) {
            edtContact.setError("Enter a valid 10-digit contact number");
            return;
        }

        int quantity = Integer.parseInt(quantityStr);
        double price = Double.parseDouble(priceStr);
        double totalAmount = quantity * price;
        double gst = totalAmount * 0.18; // 18% GST
        double finalAmount = totalAmount + gst;

        // Send data to BillActivity
        Intent intent = new Intent(MainActivity.this, BillActivity.class);
        intent.putExtra("customerName", customerName);
        intent.putExtra("address", address);
        intent.putExtra("city", city);
        intent.putExtra("contact", contact);
        intent.putExtra("medicine", medicine);
        intent.putExtra("quantity", quantity);
        intent.putExtra("price", price);
        intent.putExtra("totalAmount", totalAmount);
        intent.putExtra("gst", gst);
        intent.putExtra("finalAmount", finalAmount);
        startActivity(intent);
    }
}

==================================================================================================================================================================
activity_bill.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:id="@+id/txt_bill"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Bill will be displayed here"
        android:textSize="18sp"
        android:textStyle="bold"
        android:padding="8dp"/>
</LinearLayout>

==================================================================================================================================================================
activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">

        <EditText
            android:id="@+id/edt_customer_name"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Customer Name"
            android:inputType="textPersonName"
            android:padding="10dp"/>

        <EditText
            android:id="@+id/edt_address"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Address"
            android:inputType="textPostalAddress"
            android:padding="10dp"/>

        <AutoCompleteTextView
            android:id="@+id/auto_city"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="City"
            android:inputType="text"
            android:padding="10dp"/>

        <EditText
            android:id="@+id/edt_contact"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Contact No"
            android:inputType="phone"
            android:maxLength="10"
            android:padding="10dp"/>

        <Spinner
            android:id="@+id/spinner_medicine"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>

        <EditText
            android:id="@+id/edt_quantity"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Quantity"
            android:inputType="number"
            android:padding="10dp"/>

        <EditText
            android:id="@+id/edt_price"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Price Per Unit"
            android:inputType="numberDecimal"
            android:padding="10dp"/>

        <Button
            android:id="@+id/btn_generate_bill"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Generate Bill"
            android:padding="12dp"
            android:backgroundTint="@android:color/holo_blue_dark"
            android:textColor="@android:color/white"
            android:textStyle="bold"
            android:layout_marginTop="16dp"/>
    </LinearLayout>
</ScrollView>
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
mainactivity.java
package com.example.employeeraise;

import android.os.Bundle;
import android.view.View;
import android.widget.ArrayAdapter;
import android.widget.AutoCompleteTextView;
import android.widget.Button;
import android.widget.EditText;
import android.widget.RadioButton;
import android.widget.RadioGroup;
import android.widget.Spinner;
import android.widget.TextView;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    private EditText edtEmployeeName, edtEmail, edtTimeTaken, edtSalary;
    private AutoCompleteTextView autoCity;
    private Spinner spinnerDepartment;
    private RadioGroup genderGroup;
    private TextView txtResult;
    private Button btnCalculate;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Initialize UI elements
        edtEmployeeName = findViewById(R.id.edt_employee_name);
        edtEmail = findViewById(R.id.edt_email);
        edtTimeTaken = findViewById(R.id.edt_time_taken);
        edtSalary = findViewById(R.id.edt_salary);
        autoCity = findViewById(R.id.auto_city);
        spinnerDepartment = findViewById(R.id.spinner_department);
        genderGroup = findViewById(R.id.gender_group);
        txtResult = findViewById(R.id.txt_result);
        btnCalculate = findViewById(R.id.btn_calculate);

        // Setup City AutoCompleteTextView
        String[] cities = {"Surat", "Ahmedabad", "Vadodara", "Rajkot", "Mumbai", "Delhi"};
        ArrayAdapter<String> cityAdapter = new ArrayAdapter<>(this, android.R.layout.simple_dropdown_item_1line, cities);
        autoCity.setAdapter(cityAdapter);

        // Setup Department Spinner
        String[] departments = {"HR", "IT", "Finance", "Sales", "Marketing"};
        ArrayAdapter<String> departmentAdapter = new ArrayAdapter<>(this, android.R.layout.simple_spinner_dropdown_item, departments);
        spinnerDepartment.setAdapter(departmentAdapter);

        // Calculate Button Click Event
        btnCalculate.setOnClickListener(v -> calculateRaise());
    }

    private void calculateRaise() {
        String name = edtEmployeeName.getText().toString().trim();
        String email = edtEmail.getText().toString().trim();
        String city = autoCity.getText().toString().trim();
        String department = spinnerDepartment.getSelectedItem().toString();
        String timeTakenStr = edtTimeTaken.getText().toString().trim();
        String salaryStr = edtSalary.getText().toString().trim();
        int selectedGenderId = genderGroup.getCheckedRadioButtonId();
        RadioButton selectedGender = findViewById(selectedGenderId);
        String gender = selectedGender != null ? selectedGender.getText().toString() : "Not Specified";

        // Validation
        if (name.isEmpty() || email.isEmpty() || city.isEmpty() || timeTakenStr.isEmpty() || salaryStr.isEmpty()) {
            Toast.makeText(this, "Please fill all details", Toast.LENGTH_SHORT).show();
            return;
        }
        if (!email.contains("@")) {
            edtEmail.setError("Enter a valid email");
            return;
        }

        int timeTaken = Integer.parseInt(timeTakenStr);
        double salary = Double.parseDouble(salaryStr);
        String efficiency = "";
        double newSalary = salary;

        // Calculate Raise Based on Efficiency
        if (timeTaken >= 2 && timeTaken <= 4) {
            efficiency = "Highly Efficient";
            newSalary += salary * 0.30; // 30% raise
        } else if (timeTaken > 4 && timeTaken <= 6) {
            efficiency = "Average Efficient";
            newSalary += salary * 0.10; // 10% raise
        } else if (timeTaken > 6 && timeTaken <= 8) {
            efficiency = "Needs Training for Efficiency Improvement";
            newSalary += 2000; // Bonus of 2000
        } else if (timeTaken > 8) {
            efficiency = "Poor Efficiency";
        }

        // Display Result
        String result = "Employee Name: " + name +
                "\nGender: " + gender +
                "\nCity: " + city +
                "\nDepartment: " + department +
                "\nTime Taken: " + timeTaken + " Hours" +
                "\nEfficiency: " + efficiency +
                "\nUpdated Salary: ₹" + newSalary;

        txtResult.setText(result);
    }
}
==============================================================================================================================================
activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">

        <!-- Employee Name -->
        <EditText
            android:id="@+id/edt_employee_name"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Employee Name"
            android:inputType="textPersonName"
            android:padding="10dp"/>

        <!-- Email ID -->
        <EditText
            android:id="@+id/edt_email"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Email ID"
            android:inputType="textEmailAddress"
            android:padding="10dp"
            android:layout_marginTop="8dp"/>

        <!-- City (AutoComplete) -->
        <AutoCompleteTextView
            android:id="@+id/auto_city"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="City"
            android:inputType="text"
            android:padding="10dp"
            android:layout_marginTop="8dp"/>

        <!-- Gender Selection -->
        <RadioGroup
            android:id="@+id/gender_group"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginTop="8dp">

            <RadioButton
                android:id="@+id/radio_male"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Male"/>

            <RadioButton
                android:id="@+id/radio_female"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Female"/>
        </RadioGroup>

        <!-- Department (Spinner) -->
        <Spinner
            android:id="@+id/spinner_department"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"/>

        <!-- Time Taken (in Hours) -->
        <EditText
            android:id="@+id/edt_time_taken"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Time Taken (in Hours)"
            android:inputType="number"
            android:padding="10dp"
            android:layout_marginTop="8dp"/>

        <!-- Basic Salary -->
        <EditText
            android:id="@+id/edt_salary"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Basic Salary"
            android:inputType="numberDecimal"
            android:padding="10dp"
            android:layout_marginTop="8dp"/>

        <!-- Calculate Button -->
        <Button
            android:id="@+id/btn_calculate"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Calculate Raise"
            android:backgroundTint="@android:color/holo_blue_dark"
            android:textColor="@android:color/white"
            android:textStyle="bold"
            android:padding="12dp"
            android:layout_marginTop="16dp"/>

        <!-- Result Display -->
        <TextView
            android:id="@+id/txt_result"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textSize="16sp"
            android:textStyle="bold"
            android:padding="10dp"
            android:layout_marginTop="16dp"/>
    </LinearLayout>
</ScrollView>
==============================================================================================================================================
androidmanifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.employeeraise"
    xmlns:tools="http://schemas.android.com/tools">

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Employeeraise"
        tools:targetApi="31">

        <!-- MainActivity: Entry point of the app -->
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:label="Employee Raise">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

    </application>
</manifest>
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
datepicker , timeoicker,alert dailog , custom dailog
DailogActivity.java
    package com.example.dailogsystem;

import android.app.AlertDialog;
import android.app.DatePickerDialog;
import android.app.TimePickerDialog;
import android.content.DialogInterface;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.DatePicker;
import android.widget.EditText;
import android.widget.TimePicker;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

import java.util.Calendar;

public class MainActivity extends AppCompatActivity {

    EditText txtDate, txtTime;
    Button btnShowDialog, btnCustomDialog;
    int mYear, mMonth, mDate, mHour, mMinute;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_dialog);
        ControlInitialization();
        EventListener();

        Calendar c = Calendar.getInstance();
        mYear = c.get(Calendar.YEAR);
        mMonth = c.get(Calendar.MONTH);
        mDate = c.get(Calendar.DAY_OF_MONTH);

        mHour = c.get(Calendar.HOUR_OF_DAY);
        mMinute = c.get(Calendar.MINUTE);

        txtDate.setText(mDate + "/" + (mMonth + 1) + "/" + mYear);
        txtTime.setText(mHour + ":" + mMinute);
    }

    private void EventListener() {
        txtTime.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                TimePickerDialog timePickerDialog = new TimePickerDialog(MainActivity.this,
                        new TimePickerDialog.OnTimeSetListener() {
                            @Override
                            public void onTimeSet(TimePicker timePicker, int hour, int minute) {
                                txtTime.setText(hour + ":" + minute);
                            }
                        }, mHour, mMinute, false);
                timePickerDialog.show();
            }
        });

        txtDate.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                DatePickerDialog datePickerDialog = new DatePickerDialog(MainActivity.this,
                        new DatePickerDialog.OnDateSetListener() {
                            @Override
                            public void onDateSet(DatePicker datePicker, int year, int month, int day) {
                                txtDate.setText(day + "/" + (month + 1) + "/" + year);
                            }
                        }, mYear, mMonth, mDate);
                datePickerDialog.show();
            }
        });

        btnShowDialog.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                AlertDialog.Builder builder = new AlertDialog.Builder(MainActivity.this);

                // Ensure that you have 'info' icon in 'res/drawable/' folder
                builder.setIcon(R.drawable.info);
                builder.setTitle("Alert Dialog Example");
                builder.setMessage("Do you want to exit?");
                builder.setCancelable(false);

                builder.setPositiveButton("Yes", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialogInterface, int i) {
                        finish();
                    }
                });

                builder.setNegativeButton("No", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialogInterface, int i) {
                        dialogInterface.cancel();
                    }
                });

                AlertDialog dialog = builder.create();
                dialog.show();
            }
        });

        btnCustomDialog.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                AlertDialog.Builder builder = new AlertDialog.Builder(MainActivity.this);
                View customView = getLayoutInflater().inflate(R.layout.custom_dialog, null);
                builder.setView(customView);

                EditText txtName = customView.findViewById(R.id.txtName);

                builder.setPositiveButton("OK", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialogInterface, int i) {
                        sendData(txtName.getText().toString());
                        dialogInterface.dismiss();
                    }
                });

                AlertDialog dialog = builder.create();
                dialog.show();
            }
        });
    }

    private void sendData(String name) {
        Toast.makeText(getApplicationContext(), "Name is: " + name, Toast.LENGTH_LONG).show();
    }

    private void ControlInitialization() {
        txtDate = findViewById(R.id.txtDate);
        txtTime = findViewById(R.id.txtTime);
        btnShowDialog = findViewById(R.id.btnShowDialog);
        btnCustomDialog = findViewById(R.id.btnCustomDialog);
    }
}

==============================================================================================================================================
customdailog.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Enter Your Name"
        android:textSize="18sp"
        android:textStyle="bold"
        android:paddingBottom="8dp"/>

    <EditText
        android:id="@+id/txtName"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="Name" />

    <Button
        android:id="@+id/btnOk"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="OK"
        android:backgroundTint="@color/black"
        android:textColor="@android:color/white"
        android:textStyle="bold"
        android:padding="10dp"
        android:layout_marginTop="12dp"/>
</LinearLayout>


==============================================================================================================================================
activity_dailog.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <!-- Date Picker -->
    <EditText
        android:id="@+id/txtDate"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Select Date"
        android:focusable="false"
        android:drawableEnd="@android:drawable/ic_menu_today"
        android:padding="10dp"/>

    <!-- Time Picker -->
    <EditText
        android:id="@+id/txtTime"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Select Time"
        android:focusable="false"
        android:drawableEnd="@android:drawable/ic_menu_recent_history"
        android:padding="10dp"
        android:layout_marginTop="8dp"/>

    <!-- Alert Dialog Button -->
    <Button
        android:id="@+id/btnShowDialog"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Show Alert Dialog"
        android:backgroundTint="@android:color/holo_blue_dark"
        android:textColor="@android:color/white"
        android:textStyle="bold"
        android:padding="12dp"
        android:layout_marginTop="16dp"/>

    <!-- Custom Dialog Button -->
    <Button
        android:id="@+id/btnCustomDialog"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Show Custom Dialog"
        android:backgroundTint="@android:color/holo_green_dark"
        android:textColor="@android:color/white"
        android:textStyle="bold"
        android:padding="12dp"
        android:layout_marginTop="8dp"/>
</LinearLayout>
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++













