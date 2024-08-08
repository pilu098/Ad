# Student:
# login.php

<?php
session_start();

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $username = trim($_POST['username']);
    $password = trim($_POST['password']);

    if ($username === "admin" && $password === "password") {
        $_SESSION['loggedin'] = true;
        header("Location: enrolment.php");
        exit();
    } else {
        $error = "Invalid username or password";
    }
}
?>

<body>
    <form method="post" action="login.php">
        <h2>Login Page</h2>
        <label for="username">Username:</label>
        <input type="text" id="username" name="username" required><br><br>

        <label for="password">Password:</label>
        <input type="password" id="password" name="password" required><br><br>

        <input type="submit" value="Login"><br><br>

        <?php if (isset($error)) { ?>
            <p class="error"><?php echo htmlspecialchars($error); ?></p>
        <?php } ?>
    </form>
</body>
</html>


==========================================================================================
# enrolment.php

<?php
session_start();
if (!isset($_SESSION['loggedin'])) {
    header("Location: login.php");
    exit();
}

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $name = $_POST['name'];
    $stream = $_POST['stream'];
    $hsc_percentage = $_POST['hsc_percentage'];
    $course = $_POST['course'];

    if (preg_match("/^[A-Za-z\s]+$/", $name) &&
        is_numeric($hsc_percentage) && $hsc_percentage >= 0 && $hsc_percentage <= 100 &&
        !empty($stream) && !empty($course)) {

        $_SESSION['student'] = [
            'name' => $name,
            'stream' => $stream,
            'hsc_percentage' => $hsc_percentage,
            'course' => $course
        ];

        $eligible = false;
        $status = "Not Eligible";

        if ($course == "Integrated M.Sc.(IT)") {
            if (($stream == "Commerce" && $hsc_percentage >= 65) ||
                ($stream == "Science" && $hsc_percentage >= 60)) {
                $eligible = true;
                $status = "Eligible";
            }
        } else {
            $status = "Admission only for Integrated M.Sc.(IT).";
        }

        $_SESSION['status'] = $status;
        header("Location: status.php");
        exit();
    } else {
        $error = "Please fill out all fields correctly.";
    }
}
?>

<!DOCTYPE html>
<html>
<head>
    <title>Student Enrolment</title>
</head>
<body>
    <h2>Student Enrolment</h2>
    <form method="post" action="enrolment.php">
        Student Name: <input type="text" name="name" required pattern="[A-Za-z\s]+"><br>
        Stream:
        <select name="stream" required>
            <option value="">Select Stream</option>
            <option value="Science">Science</option>
            <option value="Commerce">Commerce</option>
            <option value="Arts">Arts</option>
        </select><br>
        HSC Percentage: <input type="number" name="hsc_percentage" required min="0" max="100"><br>
        Course:
        <select name="course" required>
            <option value="">Select Course</option>
            <option value="Integrated M.Sc.(IT)">Integrated M.Sc.(IT)</option>
            <option value="B.Com">B.Com</option>
            <option value="B.A">B.A</option>
        </select><br>
        <input type="submit" value="submit">
    </form>
    <?php
    if (isset($error)) {
        echo "<p style='color:red;'>$error</p>";
    }
    ?>
</body>
</html>

===========================================================================================
# status.php
<?php
session_start();
if (!isset($_SESSION['loggedin']) || !isset($_SESSION['student'])) {
    header("Location: login.php");
    exit();
}

$student = $_SESSION['student'];
$status = $_SESSION['status'];
?>

<html>
<head>
    <title>Admission Status</title>
</head>
<body>
    <h2>Admission Status</h2>
    <p>Student Name: <?php echo htmlspecialchars($student['name']); ?></p>
    <p>Stream: <?php echo htmlspecialchars($student['stream']); ?></p>
    <p>HSC Percentage: <?php echo htmlspecialchars($student['hsc_percentage']); ?></p>
    <p>Course: <?php echo htmlspecialchars($student['course']); ?></p>
    <p>Status: <?php echo htmlspecialchars($status); ?></p>
    <p><a href="enrolment.php">Back to Enrolment</a></p>
</body>
</html>

======================================================================================================

# Employee
# login.php
<?php


session_start();
$username = "admin";
$password = "password";

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    if ($_POST['username'] === $username && $_POST['password'] === $password) {
        $_SESSION['loggedin'] = true;
        header("Location: employee_info.php");
        exit;
    } else {
        $error = "Invalid username or password";
    }
}
?>

<body>
    <div class="container">
        <h2>Login</h2>
        <?php if (!empty($error)): ?>
            <p class="error"><?= $error ?></p>
        <?php endif; ?>
        <form method="post" action="login.php">
            <label>Username:</label>
            <input type="text" name="username" required><br><br>
            <label>Password:</label>
            <input type="password" name="password" required><br><br>
            <button type="submit">Login</button>
        </form>
    </div>
</body>
</html>

=======================================================================================

# employee_info.php

<?php
// employee_info.php

session_start();
if (!isset($_SESSION['loggedin'])) {
    header("Location: login.php");
    exit;
}

$errors = [];
$employee = [];

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    // Validate Employee Code
    if (!is_numeric($_POST['emp_code'])) {
        $errors['emp_code'] = "Employee Code must be numeric.";
    } else {
        $employee['emp_code'] = $_POST['emp_code'];
    }

    // Validate Name
    if (!ctype_alpha(str_replace(' ', '', $_POST['name']))) {
        $errors['name'] = "Name should contain only alphabets.";
    } else {
        $employee['name'] = $_POST['name'];
    }

    // Validate Department
    if (!ctype_alpha(str_replace(' ', '', $_POST['department']))) {
        $errors['department'] = "Department should contain only alphabets.";
    } else {
        $employee['department'] = $_POST['department'];
    }

    // Validate Designation
    if (!ctype_alpha(str_replace(' ', '', $_POST['designation']))) {
        $errors['designation'] = "Designation should contain only alphabets.";
    } else {
        $employee['designation'] = $_POST['designation'];
    }

    // Validate Basic Salary
    if (!is_numeric($_POST['basic_salary'])) {
        $errors['basic_salary'] = "Basic Salary must be numeric.";
    } else {
        $employee['basic_salary'] = $_POST['basic_salary'];
    }

    if (empty($errors)) {
        // Calculate salary components
        $bs = $employee['basic_salary'];
        $da = 1.25 * $bs;
        $ma = 0.1 * $bs;
        $pf = 0.13 * $bs;
        $tax = 0.1 * $bs;
        $hra = 0; // Assuming HRA is not mentioned, setting it to zero
        $gross_salary = $bs + $hra + $da + $ma;
        $net_salary = $gross_salary - $pf - $tax;

        // Store details in session
        $_SESSION['employee'] = [
            'name' => $employee['name'],
            'da' => $da,
            'ma' => $ma,
            'pf' => $pf,
            'tax' => $tax,
            'gross_salary' => $gross_salary,
            'net_salary' => $net_salary,
        ];

        header("Location: view_salary.php");
        exit;
    }
}
?>
    <div class="container">
        <h2>Employee Information</h2>
        <form method="post" action="employee_info.php">
            <label>Employee Code:</label><br>
            <input type="text" name="emp_code" value="<?= htmlspecialchars($_POST['emp_code'] ?? '') ?>"><br>
            <?php if (isset($errors['emp_code'])): ?><p class="error"><?= $errors['emp_code'] ?></p><?php endif; ?><br>

            <label>Name:</label><br>
            <input type="text" name="name" value="<?= htmlspecialchars($_POST['name'] ?? '') ?>"><br>
            <?php if (isset($errors['name'])): ?><p class="error"><?= $errors['name'] ?></p><?php endif; ?><br>

            <label>Department:</label><br>
            <input type="text" name="department" value="<?= htmlspecialchars($_POST['department'] ?? '') ?>"><br>
            <?php if (isset($errors['department'])): ?><p class="error"><?= $errors['department'] ?></p><?php endif; ?><br>

            <label>Designation:</label><br>
            <input type="text" name="designation" value="<?= htmlspecialchars($_POST['designation'] ?? '') ?>"><br>
            <?php if (isset($errors['designation'])): ?><p class="error"><?= $errors['designation'] ?></p><?php endif; ?><br>

            <label>Basic Salary:</label><br>
            <input type="text" name="basic_salary" value="<?= htmlspecialchars($_POST['basic_salary'] ?? '') ?>"><br>
            <?php if (isset($errors['basic_salary'])): ?><p class="error"><?= $errors['basic_salary'] ?></p><?php endif; ?><br>

            <br><button type="submit">Submit</button>
        </form>
    </div>
</body>
</html>


=============================================================================================
# view_salary.php

<?php

session_start();
if (!isset($_SESSION['loggedin']) || !isset($_SESSION['employee'])) {
    header("Location: login.php");
    exit;
}

$employee = $_SESSION['employee'];
?>
    <body>
        <div class="container">
            <h2>Salary Details</h2>
            <table>
                <tr>
                    <th>Name</th>
                    <td><?= $employee['name'] ?></td>
                </tr>
                <tr>
                    <th>DA</th>
                    <td><?= $employee['da'] ?></td>
                </tr>
                <tr>
                    <th>MA</th>
                    <td><?= $employee['ma'] ?></td>
                </tr>
                <tr>
                    <th>PF</th>
                    <td><?= $employee['pf'] ?></td>
                </tr>
                <tr>
                    <th>Tax</th>
                    <td><?= $employee['tax'] ?></td>
                </tr>
                <tr>
                    <th>Gross Salary</th>
                    <td><?= $employee['gross_salary'] ?></td>
                </tr>
                <tr>
                    <th>Net Salary</th>
                    <td><?= $employee['net_salary'] ?></td>
                </tr>
            </table>
        </div>
    </body>
</html>



