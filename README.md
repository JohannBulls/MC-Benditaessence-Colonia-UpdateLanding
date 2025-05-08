# MC-Benditaessence-Colonia-UpdateLanding

## 🧾 **Form Documentation with AMPscript and HTML - Salesforce Marketing Cloud**

### 📄 **Overview**

This form collects user data and updates it in a **Data Extension (DE)** of Salesforce Marketing Cloud called `PRUEBA_ACTU_DATOS`. If an ID already exists, it retrieves the missing data and updates the corresponding fields depending on the type of identification entered. Finally, it redirects to a thank-you page if the update is successful.

This form is sent to users through a **Journey** in Marketing Cloud. The WhatsApp button sends the landing page URL with the format `url/?Id=%%Id%%`. The user must fill in all the fields before submitting the form. If any data is missing, the form will not allow submission.


## 🔧 **AMPscript**

### 🟦 **Retrieve form parameters**

```ampscript
set @id = RequestParameter("Id")
set @store = RequestParameter("Store")
set @firstName = RequestParameter("FirstName")
set @lastName = RequestParameter("LastName")
set @tipoIdentificacion = RequestParameter("TipoIdentificacion")
set @identidad = RequestParameter("identidad")
set @pasaporte = RequestParameter("pasaporte")
set @phone = RequestParameter("Phone")
set @email = RequestParameter("Email")
set @contactPref = RequestParameter("ContactPreference")
set @cashBack = RequestParameter("CashBack")
set @fecha = RequestParameter("PersonBirthdate")
set @redirectURL = ""
```

### 📥 **Retrieve missing data from the Data Extension**

```ampscript
if empty(@store) and not empty(@id) then
  set @store = Lookup("PRUEBA_ACTU_DATOS", "Store", "Id", @id)
endif

if empty(@cashBack) and not empty(@id) then
  set @cashBack = Lookup("PRUEBA_ACTU_DATOS", "CashBack", "Id", @id)
endif

if empty(@fecha) and not empty(@id) then
  set @fecha = Lookup("PRUEBA_ACTU_DATOS", "PersonBirthdate", "Id", @id)
endif
```


### ✍️ **Validate and update data**

#### Step 1: **Verify required fields**

```ampscript
if not empty(@id) and not empty(@firstName) and not empty(@lastName) and not empty(@tipoIdentificacion) and not empty(@phone) and not empty(@email) and not empty(@contactPref) and not empty(@cashBack) then
```
This AMPscript condition checks if all required fields are filled. It ensures that fields like @id, @firstName, @lastName, @tipoIdentificacion, @phone, @email, @contactPref, and @cashBack are not empty. If any of these fields are empty, the script won't continue. If all fields are filled, the script proceeds with the next steps, such as updating the Data Extension and redirecting the user.

This validation ensures that only complete data is processed.
#### Step 2: **Validate identification type**

```ampscript
if @tipoIdentificacion == "Identidad" and not empty(@identidad) then
```

#### Step 3: **Update Data Extension for "Identidad"**

```ampscript
set @update = UpdateDE("PRUEBA_ACTU_DATOS", 1, "Id", @id,
  "Store", @store,
  "FirstName", @firstName,
  "LastName", @lastName,
  "TipoIdentificacion", @tipoIdentificacion,
  "identidad", @identidad,
  "pasaporte", "",
  "Phone", @phone,
  "Email", @email,
  "ContactPreference", @contactPref,
  "CashBack", @cashBack,
  "FechaCreacion", @fecha
)
```

#### Step 4: **Set redirect URL**

```ampscript
set @redirectURL = "https://cloud.comunicaciones.lacolonia.com/Gracias02"
```

#### Step 5: **Validate passport identification type**

```ampscript
elseif @tipoIdentificacion == "Pasaporte" and not empty(@pasaporte) then
```

#### Step 6: **Update Data Extension for "Pasaporte"**

```ampscript
set @update = UpdateDE("PRUEBA_ACTU_DATOS", 1, "Id", @id,
  "Store", @store,
  "FirstName", @firstName,
  "LastName", @lastName,
  "TipoIdentificacion", @tipoIdentificacion,
  "identidad", "",
  "pasaporte", @pasaporte,
  "Phone", @phone,
  "Email", @email,
  "ContactPreference", @contactPref,
  "CashBack", @cashBack,
  "FechaCreacion", @fecha
)
```

#### Step 7: **Redirect to the thank-you page**

```ampscript
set @redirectURL = "https://cloud.comunicaciones.lacolonia.com/Gracias02"
```

#### Step 8: **Close validation**

```ampscript
endif
```

#### Step 9: **Close all parameter validation**

```ampscript
endif
```



### 🔁 **Redirection**

```ampscript
if not empty(@redirectURL) then
  redirect(@redirectURL)
endif
```



## 🖼️ **HTML Form**

### **Main form fields**

* First Name
* Last Name
* Type of Identification (Identity / Passport)
* Identification or Passport Number
* Phone
* Email
* Contact Preference (Email / SMS)
* Do you want cashback? (Yes / No)

### **Hidden fields**

```html
<input type="hidden" name="Store" value="%%=v(@store)=%%">
<input type="hidden" name="Id" value="%%=v(@id)=%%">
<input type="hidden" name="FechaCreacion" value="%%=FormatDate(Now(), "yyyy-MM-dd")=%%">
```



## 💡 **JavaScript**

### **Show dynamic fields based on identification type**

```javascript
function showIdentificationField() {
  const tipo = document.getElementById("TipoIdentificacion").value;
  const identityField = document.getElementById("campoIdentidad");
  const passportField = document.getElementById("campoPasaporte");

  if (tipo === "Identidad") {
    identityField.style.display = "block";
    passportField.style.display = "none";
    document.getElementById("identidad").setAttribute("required", "true");
    document.getElementById("pasaporte").removeAttribute("required");
  } else if (tipo === "Pasaporte") {
    passportField.style.display = "block";
    identityField.style.display = "none";
    document.getElementById("pasaporte").setAttribute("required", "true");
    document.getElementById("identidad").removeAttribute("required");
  } else {
    identityField.style.display = "none";
    passportField.style.display = "none";
    document.getElementById("pasaporte").removeAttribute("required");
    document.getElementById("identidad").removeAttribute("required");
  }
}
```

### **Change browser tab icon**

```javascript
<script>
  var link = document.createElement('link');
  link.type = 'image/png';
  link.rel = 'icon';
  link.href = 'https://image.comunicaciones.lacolonia.com/lib/fe3411727364047e731772/m/1/b7928d52-9515-4d9e-a7e7-b68d5bd26ac5.png';
  document.getElementsByTagName('head')[0].appendChild(link);
</script>
```



## ✅ **Process Flow**

1. The user receives a WhatsApp message with a link to the landing page.
2. By clicking on the link, the URL contains the user's ID (`url/?Id=%%Id%%`).
3. The user accesses the page and fills out the form with the required data.
4. If any required fields are missing, the form will not allow submission.
5. Once all fields are completed, the information is submitted, and the Data Extension is updated.
6. The user is redirected to a thank-you page.



## 📌 **Final Notes**

* The DE `PRUEBA_ACTU_DATOS` must have columns with the exact names used in `UpdateDE()`.
* The form is styled with Bootstrap and is responsive.
* The creation date field is automatically generated at the time of submission.

---

### ✨ Additional Notes to Keep in Mind ✨

* The form utilizes the **Bootstrap** library, which has been preloaded in **Salesforce Marketing Cloud** within the CloudPage. 📦
* The necessary Bootstrap files are included in the page with the following links:

  ```html
  <script src="https://cloud.comunicaciones.lacolonia.com/bootstrap.bundle.min.js"></script>
  <link href="https://cloud.comunicaciones.lacolonia.com/bootstrap.min.css" rel="stylesheet">
  ```
* **`bootstrap.bundle.min.js`**: Contains the required Bootstrap JavaScript components for interactive elements like modals and form validations. ⚙️
* **`bootstrap.min.css`**: Provides the necessary CSS styles to ensure that the form is responsive and visually appealing on various devices. 📱💻

This setup ensures the form is **responsive** and leverages Bootstrap’s functionality for an enhanced user experience. 🌟

In addition, there are custom styles included to override the default styles of Salesforce Marketing Cloud to ensure that the Bootstrap styles are applied correctly. Without these styles, the Marketing Cloud default styles would conflict with Bootstrap. 
These styles ensure that the form layout and elements are properly displayed and behave as expected across different screen sizes. 📱💻

---
© Developed by Johann Amaya | 2025
