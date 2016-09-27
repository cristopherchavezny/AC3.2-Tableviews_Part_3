# C4Q Movie Reel 

### (AC3.2-Tableviews_Part_3: **Tableview Segues**)

---

### Time to Reel it In

Reel Good took our latest prototype and presented it to their board of investors as the cornerstone of their mobile initiative. The extra work we put behind the design of the app must have really sold it, because Reel Good now needs one more set of changes before they take the app into beta. 

For the next [MVP](https://www.quora.com/What-is-a-minimum-viable-product/answer/Suren-Samarchyan?srid=dpgi), Reel Good wants to be able to tap on the movie cell to display full details of the movie on a different screen. 

After the meeting with Reel Good, we sat down with our engineering team and sketched out the necessary requirements of this next task and broke them down:

1. We're going to have to implement addition `UITableviewDelegate` methods to be able to handle tap detection
2. We have to create a new custom view controller to present a full screen version of the movie data
  3. We need to make sure that navigation works to go to and from one of these view controllers.
  4. The proper movie data has to be transferred from one view controller to the next, so we need to do some data handling
  
A large part of development is being able to translate feature requests into actual programming work. Taking some time to plan out a course of action before starting to code will likely save you some time in the end. Even then, there absolutely will be unforseen problems that you will encounter. But that's just part of the fun of programming.

---
### Goals
1. Create a new custom `UIViewController` to display a single `Movie` object's data
2. Understand segues in storyboard to transition between view controllers
3. Better understanding of the delegate pattern in programming.

---

### Readings
 1. [General Reference on Xcode (very useful)](http://help.apple.com/xcode/mac/8.0)
 2. [Configuring a Segue in Storyboard - Apple](http://help.apple.com/xcode/mac/8.0/#/deve5fc2eb19)
 3. [Using Segues (lots of great info here)](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/UsingSegues.html)
 4. [Navigation Controller Implementation - tuts+ (helpful reference and example)](https://code.tutsplus.com/tutorials/ios-from-scratch-with-swift-navigation-controllers-and-view-controller-hierarchies--cms-25462)
  
---

### 1. Adding a new `UIViewController` to Display a `Movie`

#### Storyboard Changes **(Use iPhone 6s for your simulation)**
1. Drag in a `UIViewController` into `Main.storyboard` from the *Objects Library* in the *Utilities Pane* and place it next to the `MovieTableViewController`
2. Select the prototype cell (make sure you have the actual `MovieTableViewCell` selected) and Control-Drag to the new `UIViewController`
3. On the outlet menu that pops up, select "Selection Segue > Show"
  - Note: The "Accessory Selection" is that action to perform when adding an "accessory view." An example of an accessory view is the `>` (called a "chevron") you see all the way to the right on a cell in the Mail or Messages app on an iPhone. 
  - The "Selection" type of segue refers to the action to take when the cell itself is tapped/selected.
4. Select the segue object in storyboard (the -> arrow), and in the *Attribute Inspector*, set it's identifier to `MovieDetailViewSegue`
5. Drag in a `UIImageView` into the view controller, giving it the following attributes:
  - `8pt` margins at the top, left, and right
  - A height of `240pt`
  - _Before adding these constraints, make sure the checkmark box for "Constrain to margins" is selected_
  - Switch the image view's `Content Mode` to `Aspect Fit`
5. Drag in 5 `UILabel`s below the `UIImageView` in a vertical row
  - Label them (in order): `Genre`, `Location`, `Summary` and `Summary Text`
  - Set their fonts to `Roboto Regular - 17pt`, except for `Summary Text` which will be `Roboto-Light - 14pt`
  - Set the number of lines for `Summary Text` to 0
6. Select **all** of the `UILabel`s at once, but holding down the Command (⌘) Key while clicking on them
  - Select *Pin* and set it to **8pt margins**, also making sure that the checkmark for "Constrain to margins" is selected
  - Set the *Vertical Content Hugging Priority* to 1000 for all labels except `Summary Text`. Instead, set the *Vertical Compression Resistance* of `Summary Text` to 1000
  - Your view controller should resemble: (add screen shot)
  
Being able to set constraints in this batching form is one of the nice advantages of using storyboards. 

#### Storyboard linking
1. Add a new file named `MovieDetailViewController` that subclasses `UIViewController` and place it in the correct folder and Xcode group
2. In `Main.storyboard` change the custom class of the view controller we just created to be `MovieDetailViewController`
3. Create outlets for each of the labels and the imageView. Name them:
  - `moviePosterImageView`, `genreLabel`, `locationLabel`, `summaryLabel`, and `summaryFullTextLabel`
4. Additionally, give `MovieDetailViewController` an instance variable of type `Movie`:
  - `internal var selectedMovie: Movie!`
  - This property will hold a reference to the `Movie` from the cell that was tapped.

#### Preparing for Segue
A `UINavigationController` is unique in that it manages its a _navigation stack_, which is a hierarchy of view controllers. 

You can think of each view controller being a card in a stack of cards.  When you **push** a view controller onto the stack, you're putting a new card on top of the _stack_ of cards. When you **pop** a view controller, you're taking a card off the top of the _stack_. 

> It's important to know that all view controllers currently on the stack can be accessed through the navigation controller, and thus exist in memory. 

The `prepare(for:sender:)` method is where we get things ready for displaying a new view controller that has been set up with a segue object in storyboard.

As mentioned in the documentation for [`prepare(for:sender:)`](https://developer.apple.com/reference/appkit/nssegueperforming/1409583-performsegue), the `sender` parameter refers to the object that has requested the segue. In our case, the `sender` is expected to be a `MovieTableviewCell`. But because the `sender` is defined as being `Any?`, we should do a check to confirm our assumptions. Moreover, we'll need the cell to determine which movie cell was tapped.

```swift
  // 1. check sender for the cell that was tapped
        if let tappedMovieCell: MovieTableViewCell = sender as? MovieTableViewCell {
        
        }
```

The `segue` object is an instance of `NSStoryboardSegue`, which has an instance property of `indentifier` that refers to the identifier string we gave to the segue earlier in `Main.storyboard` (we used `MovieDetailViewSegue`). In order to make sure that we have the correct segue (a storyboard can have many, many segues), we need to check that the identifier matches one that we expect:

```swift
          // 1. check sender for the cell that was tapped
        if let tappedMovieCell: MovieTableViewCell = sender as? MovieTableViewCell {
         
            // 2. check for the right storyboard segue
            if segue.identifier == "MovieDetailViewSegue" {
                
            }
        }
```

While generally you should avoid force unwrapping, in this instance because we're certain of both the sender and the storyboard segue we can say with some certainty that the `segue.destination` is going to be a `MovieDetailViewController`:

`let movieDetailViewController: MovieDetailViewController = segue.destination as! MovieDetailViewController`

Ok, so we have the tapped cell (`sender`) and we have the instance of `MovieDetailViewController` (`segue.destination`), but how do we get the `Movie` object that corresponds to the cell we selected? 

We already wrote a piece of code in `cellForRow` that arranged our `movieData` by genre, but that required having the current `indexPath`. Fortunately, there's `indexPath(for:)` function that we can call on our table view to get that same info. Using that function, along with our code from `cellForRow` we have:

```swift
          // 1. check sender for the cell that was tapped
        if let tappedMovieCell: MovieTableViewCell = sender as? MovieTableViewCell {
         
            // 2. check for the right storyboard segue
            if segue.identifier == "MovieDetailViewSegue" {
                
                // 3. get reference to the destination view controller
                let movieDetailViewController: MovieDetailViewController = segue.destination as! MovieDetailViewController
                
                // 4. get our cell's indexPath
                let cellIndexPath: IndexPath = self.tableView.indexPath(for: tappedMovieCell)!
                
                // 5. get our cell's Movie
                guard let genre = Genre.init(rawValue: cellIndexPath.section),
                  let data = byGenre(genre) else {
                  return
                }
                
                // 6. set the destionation's selectedMovie property
                let selectedMovie: Movie = data[cellIndexPath.row]
                movieDetailViewController.selectedMovie = selectedMovie
            }
        }
```

Now that the destination `MovieDetailViewController` has its `Movie` object, let's populate the labels:

```swift
        self.moviePosterImageView.image = UIImage(named: movie.poster)
        self.genreLabel.text = "Genre: " + movie.genre.capitalized
        self.locationLabel.text = "Locations: " + movie.locations.joined(separator: ", ")
        self.summaryFullTextLabel.text = movie.summary
```

Go ahead and run the poject at this point to see if the data gets passed along properly...

`fatal error: unexpectedly found nil while unwrapping an Optional value`

> **Discuss & Debug**:
Why are we getting force unwrapping errors? 

#### Updating `MovieDetailViewController` with a `Movie`

Let's move our code into `MovieDetailViewController` in a new function, `updateViews(for:)`:

```swift
    internal func updateViews(for movie: Movie) {
        self.moviePosterImageView.image = UIImage(named: movie.poster)!
        self.genreLabel.text = "Genre: " + movie.genre.capitalized
        self.locationLabel.text = "Locations: " + movie.locations.joined(separator: ", ")
        self.summaryFullTextLabel.text = movie.summary
    }
```

And lastly, let's call it in our `viewDidLoad`

```swift
    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.updateViews(for: self.selectedMovie)
    }
```

And re-run the project now. You should see (screenshot)

---




