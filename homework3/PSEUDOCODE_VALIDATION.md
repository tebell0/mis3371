# Pseudocode: Validation Logic & Validate Button
# Houston Health Center — homework3.js

---

## 1. Validation Rules (getValidations)

```
FUNCTION getValidations():
    RETURN a map of fieldName → validation function:

        first_name:       value is not empty (after trimming whitespace)
        middle_initial:   value length is 0 or 1 character (optional)
        last_name:        value is not empty (after trimming whitespace)

        dob:
            IF value does not match pattern MM/DD/YYYY → RETURN false
            PARSE month, day, year from value
            BUILD dobDate from parsed values
            IF dobDate is in the future         → RETURN false
            IF dobDate is older than 120 years  → RETURN false
            IF parsed date components don't match rebuilt date (catches invalid days/months) → RETURN false
            RETURN true

        ssn:              value matches pattern ###-##-####
        phone:            value matches pattern ###-###-####
        email:            value contains "@", a domain, and a TLD (basic regex)
        address:          value is not empty
        address2:         always RETURN true (optional field)
        city:             value is not empty
        state:            value is not empty (a state has been selected from dropdown)
        zip:              value matches exactly 5 digits
        insurance:        value is not empty
        policy_number:    value is not empty
        description:      value is not empty
        pain_level:       always RETURN true (range slider always has a value)
        age_group:        value is not empty (a radio button has been selected)
        vaccinated:       always RETURN true (optional field)
        user_id:          value is not empty

        password:
            GET first_name value from form (lowercase)
            GET last_name value from form (lowercase)
            IF password is empty                         → RETURN false
            IF password contains first_name (and first_name is not empty) → RETURN false
            IF password contains last_name  (and last_name  is not empty) → RETURN false
            RETURN true

        confirm_password:
            RETURN true IF value is not empty AND value equals the password field's value
```

---

## 2. Inline Field Error Display (setFieldError / setRadioGroupError)

```
FUNCTION setFieldError(fieldName, isValid):
    GET the input element with name = fieldName
    GET the error <span> element with id = "error_[fieldName]"
    IF either element does not exist → EXIT

    IF isValid:
        REMOVE "field-error" class from input  (clears red border/background)
        SET error span text to ""
        HIDE error span
    ELSE:
        ADD "field-error" class to input       (applies red border/background)
        SET error span text to errorMessages[fieldName]
        SHOW error span inline


FUNCTION setRadioGroupError(groupName, isValid):
    GET all radio inputs with name = groupName
    GET the error <span> element with id = "error_[groupName]"

    FOR EACH radio input:
        IF isValid → REMOVE "field-error" class
        ELSE       → ADD    "field-error" class

    IF isValid:
        SET error span text to ""
        HIDE error span
    ELSE:
        SET error span text to errorMessages[groupName]
        SHOW error span
```

---

## 3. Real-Time Validation (attachListeners / validateField)

```
FUNCTION validateField(fieldName):
    GET validations map
    GET the form element for fieldName

    IF element is a radio input:
        GET the currently checked radio in the group
        SET value = checked radio's value, OR "" if none checked
        valid = run validations[fieldName](value)
        CALL setRadioGroupError(fieldName, valid)
        RETURN valid

    ELSE:
        SET value = element's current value
        valid = run validations[fieldName](value)
        CALL setFieldError(fieldName, valid)
        RETURN valid


FUNCTION attachListeners():
    FOR EACH text/password/select/textarea field:

        ON "blur" (user leaves the field):
            CALL validateField(fieldName)
            MARK field as "touched" (dataset.touched = true)

        ON "input" (user is typing):
            IF field already has "field-error" class OR field has been touched:
                CALL validateField(fieldName)   ← live re-validation after first touch

        SPECIAL CASE — password field "input":
            IF confirm_password has been touched OR has an error:
                CALL validateField("confirm_password")
                ← keeps confirm_password in sync as password is edited

    FOR EACH radio group (age_group):
        ON "change" (user selects a radio):
            CALL validateField(groupName)
```

---

## 4. Full-Form Validation (validateAll)

```
FUNCTION validateAll():
    GET validations map
    SET allValid = true

    FOR EACH field in allFields list:
        GET form element for field

        IF element is a radio group:
            GET checked radio value (or "" if none)
            valid = run validations[fieldName](value)
            CALL setRadioGroupError(fieldName, valid)
        ELSE:
            SET value = element's current value
            valid = run validations[fieldName](value)
            CALL setFieldError(fieldName, valid)

        IF NOT valid → SET allValid = false

    RETURN allValid
```

---

## 5. Validate Button Click Handler

```
ON "validate_button" CLICK:

    GET form reference
    GET validations map
    INITIALIZE errors = empty list

    FOR EACH field in allFields list:
        GET form element for field

        IF element is a radio group:
            GET checked radio value (or "" if none)
            valid = run validations[fieldName](value)
            CALL setRadioGroupError(fieldName, valid)
        ELSE:
            SET value = element's current value
            valid = run validations[fieldName](value)
            CALL setFieldError(fieldName, valid)

        IF NOT valid:
            ADD { label, errorMessage } to errors list

    ── Scroll to first error ──
    IF errors list is not empty:
        FIND first element on the page with class "field-error"
        SCROLL that element into view (smooth, centered)

    ── Build modal content ──
    IF errors list is not empty:
        DISPLAY error modal:
            Header:  "⚠ Validation Failed"
            Summary: "[N] error(s) found. Please fix the following fields:"
            List:    FOR EACH error → show field label + specific error message
            Button:  "Close" only → dismisses modal, user fixes errors on form

    ELSE (all fields valid):
        DISPLAY success modal:
            Header:  "✓ All Fields Valid!"
            Message: "Your information looks good. Click Submit to complete your registration."
            Buttons:
                "Close"  → dismisses modal, returns to form
                "Submit" → dismisses modal, calls form.submit() → redirects to closingpage3.html
```

---

## 6. Form Reset

```
ON "reset" BUTTON CLICK:
    WAIT for browser to clear field values (setTimeout 0ms), THEN:
        FOR EACH element with class "field-error" → REMOVE "field-error" class
        FOR EACH error <span> with class "error-msg":
            SET text to ""
            HIDE span
        FOR EACH element with data-touched attribute → REMOVE attribute
        RESET pain slider value to 15
        UPDATE pain slider display label to 15
```
