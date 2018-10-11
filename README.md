# Sprint Challenge 2: Lambda Student Files

This sprint challenge reviews many techniques you've learned over the last two weeks. You'll create a full featured manager that allows you to create, delete, and modify student records.

This challenge builds off the project you built yesterday, so the starting point will seem quite similar although there are a few important differences.

This is a pretty big project but you've done everything here over the past few weeks. So if you think something is unfamiliar, take a moment and look over your work, your notes, and your projects.

Your final project will demonstrate how you can design an interface that enables you to add, edit, and remove records, which is a fundamental part of so many applications!

Good luck and believe in yourself.

## Student records

Create two new types in separate files: `Student` and `StudentManager`. These types should both be classes, even though you used a `Student` struct yesterday.  Also every property in `Student` should now be a `var` and not `let`.

As you did yesterday, conform `Student` to `CustomStringConvertible` and `Codable`. Implement `description` as you did, to create a human-readable description of the student record.

Further, as you are working with classes, add an initializer that sets up each type member property. You may default the optionals to nil if you like.

Build a Student Manager as you did yesterday. Your `StudentManager` type should be a class, descended from `NSObject`. It will store an array of students in a variable property called `students`.

As you did yesterday, implement the `writeToFile` and `readFromFile` methods, making sure you use `do`-`catch` error management for `data.write(to:)` and `Data(contentsOf:)`. You will encode to JSON data before writing and decode after reading.

```
let url = URL(fileURLWithPath: NSHomeDirectory())
    .appendingPathComponent("Documents")
    .appendingPathComponent("students.json")
    
func writeToFile() {
}    

func readFromFile() {
}
```

Add methods to add a new student record (`addRecord(_ student: Student)`), remove a student record (`func removeRecord(at idx: Int)`) and return a record at a given index (`func record(at idx: Int) -> Student`).

Unlike yesterday, have the class perform `readFromFile()` in its initializer instead of using the main view controller.

## Building the primary view controller

Create a table view controller embedded in a navigation controller as your Main storyboard entry point. Add an object and set its class to StudentManager. Connect your controller to this object with `@IBOutlet var studentManager: StudentManager!` so it can use its data to drive the application.

The table should read and load the `studentManager` data each time it appears on-screen.

Make sure you set a re-use identifier. Pull the number of rows and cell contents from the `studentManager` data.

To support "swipe to delete" finish the logic in the following table controller method:

```
override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
    
    // Only handle deletions
    guard editingStyle == .delete else { return }
    
    // Update model then refresh view
    studentManager.removeRecord(at: indexPath.row)
    studentManager.writeToFile()
    tableView.reloadData()
  }
 ```
  
## Creating the detail view controller class
 
Add a second controller class file (StudentViewController.swift) and create a new UIViewController subclass. 

Add the following outlets and actions:

```
  @IBOutlet var nameField: UITextField!
  @IBOutlet var ageField: UITextField!
  @IBOutlet var cohortField: UITextField!
  
  @IBAction func cancel(_ sender: Any) {
    dismiss(animated: true, completion: nil)
  }
```

Add a second view controller to IB. Set its type to `StudentViewController`. Embed it in a navigation controller.

Add a bar button item to your main controller. Set it to the system Add style. Have it present the student view controller modally. Name the segue as `addNewRecord`.

Connect the prototype cell to your view controller to present it modally. Name the segue `editRecord`.

Design the StudentViewController screen with two bar button items: Cancel and Save. (These are both available as system items.) Connect the cancel item to the `cancel` IBAction.

Add three labels and three fields for the Name, Age, and Cohort. Add placeholder text and use auto layout to create a pleasing layout. Your layout should work well in both landscape and portrait orientations.

Implement a `func studentRecord() -> Student?` method that creates a student from the data in these fields. If the name is empty, you must return nil. You'll need to convert from text to the integer needed for `age`.

Create a `var student: Student?` property. Implement `override func viewWillAppear(_ animated: Bool)` and have it load the data from `student` into the fields when `student` is not nil. A non-nil `student` indicates you are editing an existing student and not adding a new student.

## Handle data flow

Implement two segue methods in your main table view controller class. Do *not* use autocomplete for the `unwind` method. The autocomplete is currently broken and will give you the wrong method name.

```
@IBAction func unwindFor(_ unwindSegue: UIStoryboardSegue, towards subsequentVC: UIViewController) {
  ...
}

override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
}
```

In IB, control drag your Save item in the StudentViewController layout to its Exit (the rightmost of the three items in the bar at the top of the controller and select `unwindFor`.

In the main class (make sure you put these in the right place!), edit `prepare`. It takes two steps to get to the right student view controller, as your destination is set up to the be navigation controller:

```
    guard
      let navigationController = segue.destination as? UINavigationController,
      let destination = navigationController.topViewController as? StudentViewController
      else { return }
```

Make sure you're working with the edit identifier. Otherwise, set `student` to nil and leave the method.

Fetch the index path for the selected row and use it to fetch the correct record from your `studentManager`.  Set the `student` property to that record.

Finish by editing `unwind`. Retrieve the source view controller:

```
    guard let source = unwindSegue.source as? StudentViewController
      else { return }
```

Check whether the source has a populated `student`. This tells you whether you're adding a record or editing a record.

If adding, just add the record and write to file.

If modifying a record, copy the values from `source.studentRecord()` to your original record (from `source.student`) and modify it in-place. Then write to file.

Don't forget to reload the tableView data after making any changes!

