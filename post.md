We’ve all seen it: A collection view where see parts of the next and previous cells are visible while the collection view can scroll with paging. When it scrolls, a cell is always centred. 

What we would also like is for that behaviour to behave like the paging which can be set through pagingEnabled in both UIScrollView and UICollectionView which is its subclass. This means that as the user scolls, the next cell will snap in the center.

![Intro](http://karmadust.com/wp-content/uploads/2015/04/Center.jpg)

This is a simple but interesting exercise because of its use of an obscure function in the UICollectionViewFlowLayout class.


```swift,linenums=true
func targetContentOffsetForProposedContentOffset(_ proposedContentOffset: CGPoint,
                           withScrollingVelocity velocity: CGPoint) -> CGPoint
```
This gets called when the user lifts his finger and the collection view (which is a scroll view) starts decelerating and will return the content offset where the collection view should stop. Normally, its implemented by the superclass but here we need to override it.

As a tip, there is a way to add a custom UICollectionViewLayout to a UICollectionView from Interface Builder without losing the panel provided by the standard flow layout where you can set the size of the cell and the distance between them. This is done by not setting the layout to custom as below:

![XCode Screen](http://karmadust.com/wp-content/uploads/2015/04/Screen-Shot-2015-04-23-at-12.32.03.png)

But by changing the class on the layout outlet itself:

![XCode Screen](http://karmadust.com/wp-content/uploads/2015/04/Screen-Shot-2015-04-23-at-12.36.04.png)

The class is shown below:

```swift,linenums=true
import UIKit
 
class CenterCellCollectionViewFlowLayout: UICollectionViewFlowLayout {
    
    override func targetContentOffsetForProposedContentOffset(proposedContentOffset: CGPoint, withScrollingVelocity velocity: CGPoint) -> CGPoint {
        
        if let cv = self.collectionView {
            
            let cvBounds = cv.bounds
            let halfWidth = cvBounds.size.width * 0.5;
            let proposedContentOffsetCenterX = proposedContentOffset.x + halfWidth;
            
            if let attributesForVisibleCells = self.layoutAttributesForElementsInRect(cvBounds) as? [UICollectionViewLayoutAttributes] {
                
                var candidateAttributes : UICollectionViewLayoutAttributes?
                for attributes in attributesForVisibleCells {
                    
                    // == Skip comparison with non-cell items (headers and footers) == //
                    if attributes.representedElementCategory != UICollectionElementCategory.Cell {
                            continue
                    }
                    
                    if let candAttrs = candidateAttributes {
                        
                        let a = attributes.center.x - proposedContentOffsetCenterX
                        let b = candAttrs.center.x - proposedContentOffsetCenterX
                        
                        if fabsf(Float(a)) < fabsf(Float(b)) {
                            candidateAttributes = attributes;
                        }
                        
                    }
                    else { // == First time in the loop == //
                        
                        candidateAttributes = attributes;
                        continue;
                    }
                    
                    
                }
                
                return CGPoint(x : candidateAttributes!.center.x - halfWidth, y : proposedContentOffset.y);
                
            }
            
        }
        
        // Fallback
        return super.targetContentOffsetForProposedContentOffset(proposedContentOffset)
    }
   
}
```

The proposed offset is where the collection view would stop without our intervention. We peek into this area by finding its centre as proposedContentOffsetCenterX and examine our currently visible cells to see which one’s centre is closer to the centre of that area.

![XCode Screen](http://karmadust.com/wp-content/uploads/2015/04/Center2.jpg)

It has been suggested that another way to achieve this is by using the UIScrollViewDelegate's 

```swift,linenums=true
- (void)scrollViewWillEndDragging:(UIScrollView *)scrollView
                     withVelocity:(CGPoint)velocity
              targetContentOffset:(inout CGPoint *)targetContentOffset
```

