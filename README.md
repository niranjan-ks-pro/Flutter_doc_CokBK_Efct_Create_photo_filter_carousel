# Flutter_doc_CokBK_Efct_Create_photo_filter_carousel
 https://docs.flutter.dev/cookbook/effects/photo-filter-carousel#interactive-example

Create a photo filter carousel
==============================

1.  [Cookbook](https://docs.flutter.dev/cookbook)
2.  [Effects](https://docs.flutter.dev/cookbook/effects)
3.  [Create a photo filter carousel](https://docs.flutter.dev/cookbook/effects/photo-filter-carousel)

Everybody knows that a photo looks better with a filter. In this recipe, you build a scrollable, filter selection carousel.

The following animation shows the app's behavior:

![Photo Filter Carousel](https://docs.flutter.dev/assets/images/docs/cookbook/effects/PhotoFilterCarousel.gif)

This recipe begins with the photo and filters already in place. Filters are applied with the `color` and `colorBlendMode` properties of the [`Image`](https://api.flutter.dev/flutter/widgets/Image-class.html) widget.

[](https://docs.flutter.dev/cookbook/effects/photo-filter-carousel#add-a-selector-ring-and-dark-gradient)Add a selector ring and dark gradient
----------------------------------------------------------------------------------------------------------------------------------------------

The selected filter circle is displayed within a selector ring. Additionally, a dark gradient is behind the available filters, which helps the contrast between the filters and any photo that you choose.

Create a new stateful widget called `FilterSelector` that you'll use to implement the selector.

content_copy

```
@immutable
class FilterSelector extends StatefulWidget {
  const FilterSelector({
    super.key,
  });

  @override
  State<FilterSelector> createState() => _FilterSelectorState();
}

class _FilterSelectorState extends State<FilterSelector> {
  @override
  Widget build(BuildContext context) {
    return const SizedBox();
  }
}
```

Add the `FilterSelector` widget to the existing widget tree. Position the `FilterSelector` widget on top of the photo, at the bottom and centered.

content_copy

```
Stack(
  children: [
    Positioned.fill(
      child: _buildPhotoWithFilter(),
    ),
    const Positioned(
      left: 0.0,
      right: 0.0,
      bottom: 0.0,
      child: FilterSelector(),
    ),
  ],
),
```

Within the `FilterSelector` widget, display a selector ring on top of a dark gradient by using a `Stack` widget.

content_copy

```
class _FilterSelectorState extends State<FilterSelector> {
  static const _filtersPerScreen = 5;
  static const _viewportFractionPerItem = 1.0 / _filtersPerScreen;

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        final itemSize = constraints.maxWidth * _viewportFractionPerItem;

        return Stack(
          alignment: Alignment.bottomCenter,
          children: [
            _buildShadowGradient(itemSize),
            _buildSelectionRing(itemSize),
          ],
        );
      },
    );
  }

  Widget _buildShadowGradient(double itemSize) {
    return SizedBox(
      height: itemSize * 2 + widget.padding.vertical,
      child: const DecoratedBox(
        decoration: BoxDecoration(
          gradient: LinearGradient(
            begin: Alignment.topCenter,
            end: Alignment.bottomCenter,
            colors: [
              Colors.transparent,
              Colors.black,
            ],
          ),
        ),
        child: SizedBox.expand(),
      ),
    );
  }

  Widget _buildSelectionRing(double itemSize) {
    return IgnorePointer(
      child: Padding(
        padding: widget.padding,
        child: SizedBox(
          width: itemSize,
          height: itemSize,
          child: const DecoratedBox(
            decoration: BoxDecoration(
              shape: BoxShape.circle,
              border: Border.fromBorderSide(
                BorderSide(width: 6, color: Colors.white),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

The size of the selector circle and the background gradient depends on the size of an individual filter in the carousel called `itemSize`. The `itemSize` depends on the available width. Therefore, a `LayoutBuilder` widget is used to determine the available space, and then you calculate the size of an individual filter's `itemSize`.

The selector ring includes an `IgnorePointer` widget because when carousel interactivity is added, the selector ring shouldn't interfere with tap and drag events.

[](https://docs.flutter.dev/cookbook/effects/photo-filter-carousel#create-a-filter-carousel-item)Create a filter carousel item
------------------------------------------------------------------------------------------------------------------------------

Each filter item in the carousel displays a circular image with a color applied to the image that corresponds to the associated filter color.

Define a new stateless widget called `FilterItem` that displays a single list item.

content_copy

```
@immutable
class FilterItem extends StatelessWidget {
  const FilterItem({
    super.key,
    required this.color,
    this.onFilterSelected,
  });

  final Color color;
  final VoidCallback? onFilterSelected;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onFilterSelected,
      child: AspectRatio(
        aspectRatio: 1.0,
        child: Padding(
          padding: const EdgeInsets.all(8),
          child: ClipOval(
            child: Image.network(
              'https://docs.flutter.dev/cookbook/img-files'
              '/effects/instagram-buttons/millenial-texture.jpg',
              color: color.withOpacity(0.5),
              colorBlendMode: BlendMode.hardLight,
            ),
          ),
        ),
      ),
    );
  }
}
```

[](https://docs.flutter.dev/cookbook/effects/photo-filter-carousel#implement-the-filter-carousel)Implement the filter carousel
------------------------------------------------------------------------------------------------------------------------------

Filter items scroll to the left and right as the user drags. Scrolling requires some kind of `Scrollable` widget.

You might consider using a horizontal `ListView` widget, but a `ListView` widget positions the first element at the beginning of the available space, not at the center, where your selector ring sits.

A `PageView` widget is better suited for a carousel. A `PageView` widget lays out its children from the center of the available space and provides snapping physics. Snapping physics is what causes an item to snap to the center, no matter where the user releases a drag.

info Note: In cases where you need to customize the position of child widgets within a scrollable area, consider using a [`Scrollable`](https://api.flutter.dev/flutter/widgets/Scrollable-class.html) widget with a [`viewportBuilder`](https://api.flutter.dev/flutter/widgets/Scrollable/viewportBuilder.html), and place a [`Flow`](https://api.flutter.dev/flutter/widgets/Flow-class.html) widget inside the `viewportBuilder`. The `Flow` widget has a [delegate property](https://api.flutter.dev/flutter/widgets/Flow/delegate.html) that allows you to position child widgets wherever you want, based on the current `viewportOffset`.

Configure your widget tree to make space for the `PageView`.

content_copy

```
@override
Widget build(BuildContext context) {
  return LayoutBuilder(builder: (context, constraints) {
    final itemSize = constraints.maxWidth * _viewportFractionPerItem;

    return Stack(
      alignment: Alignment.bottomCenter,
      children: [
        _buildShadowGradient(itemSize),
        _buildCarousel(itemSize),
        _buildSelectionRing(itemSize),
      ],
    );
  });
}

Widget _buildCarousel(double itemSize) {
  return Container(
    height: itemSize,
    margin: widget.padding,
    child: PageView.builder(
      itemCount: widget.filters.length,
      itemBuilder: (context, index) {
        return const SizedBox();
      },
    ),
  );
}
```

Build each `FilterItem` widget within the `PageView` widget based on the given `index`.

content_copy

```
Color itemColor(int index) => widget.filters[index % widget.filters.length];

Widget _buildCarousel(double itemSize) {
  return Container(
    height: itemSize,
    margin: widget.padding,
    child: PageView.builder(
      itemCount: widget.filters.length,
      itemBuilder: (context, index) {
        return Center(
          child: FilterItem(
            color: itemColor(index),
            onFilterSelected: () {},
          ),
        );
      },
    ),
  );
}
```

The `PageView` widget displays all of the `FilterItem` widgets, and you can drag to the left and right. However, right now each `FilterItem` widget takes up the entire width of the screen, and each `FilterItem` widget is displayed at the same size and opacity. There should be five `FilterItem` widgets on the screen, and the `FilterItem` widgets need to shrink and fade as they move farther from the center of the screen.

The solution to both of these issues is to introduce a `PageViewController`. The `PageViewController`'s `viewportFraction` property is used to display multiple `FilterItem` widgets on the screen at the same time. Rebuilding each `FilterItem` widget as the `PageViewController` changes allows you to change each `FilterItem` widget's size and opacity as the user scrolls.

Create a `PageViewController` and connect it to the `PageView` widget.

content_copy

```
class _FilterSelectorState extends State<FilterSelector> {
  static const _filtersPerScreen = 5;
  static const _viewportFractionPerItem = 1.0 / _filtersPerScreen;

  late final PageController _controller;

  Color itemColor(int index) => widget.filters[index % widget.filters.length];

  @override
  void initState() {
    super.initState();
    _controller = PageController(
      viewportFraction: _viewportFractionPerItem,
    );
    _controller.addListener(_onPageChanged);
  }

  void _onPageChanged() {
    final page = (_controller.page ?? 0).round();
    widget.onFilterChanged(widget.filters[page]);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  Widget _buildCarousel(double itemSize) {
    return Container(
      height: itemSize,
      margin: widget.padding,
      child: PageView.builder(
        controller: _controller,
        itemCount: widget.filters.length,
        itemBuilder: (context, index) {
          return Center(
            child: FilterItem(
              color: itemColor(index),
              onFilterSelected: () {},
            ),
          );
        },
      ),
    );
  }
}
```

With the `PageViewController` added, five `FilterItem` widgets are visible on the screen at the same time, and the photo filter changes as you scroll, but the `FilterItem` widgets are still the same size.

Wrap each `FilterItem` widget with an `AnimatedBuilder` to change the visual properties of each `FilterItem` widget as the scroll position changes.

content_copy

```
Widget _buildCarousel(double itemSize) {
  return Container(
    height: itemSize,
    margin: widget.padding,
    child: PageView.builder(
      controller: _controller,
      itemCount: widget.filters.length,
      itemBuilder: (context, index) {
        return Center(
          child: AnimatedBuilder(
            animation: _controller,
            builder: (context, child) {
              return FilterItem(
                color: itemColor(index),
                onFilterSelected: () => {},
              );
            },
          ),
        );
      },
    ),
  );
}
```

The `AnimatedBuilder` widget rebuilds every time the `_controller` changes its scroll position. These rebuilds allow you to change the `FilterItem` size and opacity as the user drags.

Calculate an appropriate scale and opacity for each `FilterItem` widget within the `AnimatedBuilder` and apply those values.

content_copy

```
Widget _buildCarousel(double itemSize) {
  return Container(
    height: itemSize,
    margin: widget.padding,
    child: PageView.builder(
      controller: _controller,
      itemCount: widget.filters.length,
      itemBuilder: (context, index) {
        return Center(
          child: AnimatedBuilder(
            animation: _controller,
            builder: (context, child) {
              if (!_controller.hasClients ||
                  !_controller.position.hasContentDimensions) {
                // The PageViewController isn't connected to the
                // PageView widget yet. Return an empty box.
                return const SizedBox();
              }

              // The integer index of the current page,
              // 0, 1, 2, 3, and so on
              final selectedIndex = _controller.page!.roundToDouble();

              // The fractional amount that the current filter
              // is dragged to the left or right, for example, 0.25 when
              // the current filter is dragged 25% to the left.
              final pageScrollAmount = _controller.page! - selectedIndex;

              // The page-distance of a filter just before it
              // moves off-screen.
              const maxScrollDistance = _filtersPerScreen / 2;

              // The page-distance of this filter item from the
              // currently selected filter item.
              final pageDistanceFromSelected =
                  (selectedIndex - index + pageScrollAmount).abs();

              // The distance of this filter item from the
              // center of the carousel as a percentage, that is, where the selector
              // ring sits.
              final percentFromCenter =
                  1.0 - pageDistanceFromSelected / maxScrollDistance;

              final itemScale = 0.5 + (percentFromCenter * 0.5);
              final opacity = 0.25 + (percentFromCenter * 0.75);

              return Transform.scale(
                scale: itemScale,
                child: Opacity(
                  opacity: opacity,
                  child: FilterItem(
                    color: itemColor(index),
                    onFilterSelected: () => () {},
                  ),
                ),
              );
            },
          ),
        );
      },
    ),
  );
}
```

Each `FilterItem` widget now shrinks and fades away as it moves farther from the center of the screen.

Add a method to change the selected filter when a `FilterItem` widget is tapped.

content_copy

```
void _onFilterTapped(int index) {
  _controller.animateToPage(
    index,
    duration: const Duration(milliseconds: 450),
    curve: Curves.ease,
  );
}
```

Configure each `FilterItem` widget to invoke `_onFilterTapped` when tapped.

content_copy

```
FilterItem(
  color: itemColor(index),
  onFilterSelected: () => _onFilterTapped,
),
```
