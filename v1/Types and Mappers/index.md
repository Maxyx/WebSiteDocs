# Types and Mappers

LiveCharts can plot any type, even user-defined types, without losing the beauty of a strongly typed language, the concept is simple, you pass collection of generic values, then LiveCharts needs a function to pull X and Y coordinates (if it is a Cartesian chart), this function is known as a **Mapper**, the library already knows how to plot, *double*, *int*, *decimal*, *short*, *float*, *long* and also some other specially designed classes, *[ObservableValue](https://github.com/beto-rodriguez/Live-Charts/blob/master/Core/Defaults/ObservableValue.cs)*, *[ObservablePoint](https://github.com/beto-rodriguez/Live-Charts/blob/master/Core/Defaults/ObservablePoint.cs)*, *[ScatterPoint](https://github.com/beto-rodriguez/Live-Charts/blob/master/Core/Defaults/ScatterPoint.cs)*, *[DateTimePoint](https://github.com/beto-rodriguez/Live-Charts/blob/master/Core/Defaults/DateTimePoint.cs)*, *[HeatPoint](https://github.com/beto-rodriguez/Live-Charts/blob/master/Core/Defaults/HeatPoint.cs)*, *[OhclPoint](https://github.com/beto-rodriguez/Live-Charts/blob/master/Core/Defaults/OHCLPoint.cs)*,     all these classes notify the chart to update when a property changes, samples in this web site use these objects, you can always define your own type.

```
var doubleValues = new ChartValues&lt;double> { 1, 2 ,3 };
var intValues = new ChartValues&lt;int> { 1, 2 ,3 };

//the observable value class notifies the chart to update
//every time the ObservableValue.Value property changes
var observableValues = new ChartValues&lt;LiveCharts.Defaults.ObservableValue> 
{ 
    new LiveCharts.Defaults.ObservableValue(1), //initializes Value property as 1
    new LiveCharts.Defaults.ObservableValue(2),
    new LiveCharts.Defaults.ObservableValue(3)
};
```

### The cartesian system

Notice a cartesian chart requires both, X and Y coordinates, but then how is it possible that the library is able to plot a collection of primitive types?

When you are plotting a collection of primitive types, LiveCharts by default will use the index of the value in your collection, and the actual value you passed,for the next case: 

```
var myValues = new LiveCharts.ChartValues&lt;double&gt;
{
  10, //index 0
  6,  //index 1
  9,  //index 2
  2,  //index 3
  7   //index 4
}
```

The real coordinates are actually:

<div class="table-responsive">
    <table class="table table-condensed">
        <thead>
            <tr>
                <th rowspan="2" class="text-center" style="width: 34%">Value</th>
                <th colspan="2" class="text-center" style="width: 33%">Horizontal Series</th>
                <th colspan="2" class="text-center" style="width: 33%">Vertical Series</th>
            </tr>
            <tr>
                <th class="text-center">X <small class="text-muted">indexed</small></th>
                <th class="text-center">Y</th>
                <th class="text-center">X</th>
                <th class="text-center">Y <small class="text-muted">indexed</small></th>
            </tr>
        </thead>
        <tbody>
            <tr class="text-center">
                <td>10</td>
                <td>0</td>
                <td>10</td>
                <td>10</td>
                <td>0</td>
            </tr>
            <tr class="text-center">
                <td>6</td>
                <td>1</td>
                <td>6</td>
                <td>6</td>
                <td>1</td>
            </tr>
            <tr class="text-center">
                <td>9</td>
                <td>2</td>
                <td>9</td>
                <td>9</td>
                <td>2</td>
            </tr>
            <tr class="text-center">
                <td>2</td>
                <td>3</td>
                <td>2</td>
                <td>2</td>
                <td>3</td>
            </tr>
            <tr class="text-center">
                <td>7</td>
                <td>4</td>
                <td>7</td>
                <td>7</td>
                <td>4</td>
            </tr>
        </tbody>
    </table>
</div>

### Series Orientation

Notice you can configure a type according to the series orientation, the orientation is already defined for every series in the library, the **Horizontal Series** (as today version 0.9.6) are: LineSeries, ColumnSeries, StackedAreaSeries, StackedColumnSeries, OHCLSeries, Candle Series and StepLineSeries. The **Vertical Series** are: VerticalLineSeries, RowSeries, VerticalStackedAreaSeries and StackedRowSeries.

### Mappers

The orientation helps the library to decide which coordinate will be the index and which one will be the value in our collection, we use a function for every orientation, this function will pull out the *X* and *Y* coordinates, and they are known as **Mappers** checkout this simple sample:

```
// this is how the library configures the int type:

//in a case where we have a Horizontal series:
var horizontalMapper = Mappers.Xy&lt;int>()
    .X((value, index) => index)  // pull the index as our X coordinate
    .Y((value, index) => value); // pull the value we passed as the Y coordinate

//now in a case where orientation is vertical, we will:
var verticalMapper = Mappers.Xy&lt;int>()
    .X((value, index) => value)  // pull the value as our X coordinate
    .Y((value, index) => index); // pull the index as Y

// finally lets save the mapper globally,
// this way we taught LiveCharts
// what to do always we require to plot the int type
LiveCharts.Charting.For&lt;int>(horizontalMapper, SeriesOrientation.Horizontal);
LiveCharts.Charting.For&lt;int>(verticalMapper, SeriesOrientation.Vertical);
```

When do we need to define a mapper? only when you need to teach LiveCharts how to plot a custom type, this example was extended for both cases (Vertical and Horizontal), but you normally don't need to specify the orientation, take a look at this running samples, it should be intuitive and really useful in some cases: <a ng-href="/App/examples/v1/{{sms.platform}}/Constant%20Changes">Constant Changes</a> and <a ng-href="/App/examples/v1/{{sms.platform}}/Filtering%20data">Filtering Data</a>

There are many types of mappers that help you to configure every series:

```
//X and Y
var mapper = Mappers.Xy&lt;ObservablePoint>() //in this case value is of type &lt;ObservablePoint>
    .X(value => value.X) //use the X property as X
    .Y(value => value.Y); //use the Y property as Y

//X, Y and Weight
var mapper = Mappers.Bubble&lt;BubblePoint>()
                .X(value => value.X)
                .Y(value => value.Y)
                .Weight(value => value.Weight);

//Angle and Radius
var mapper = Mappers.Polar&lt;PolarPoint>()
    .Radius(value => value.Radius) //use the radius property as radius for the plotting
    .Angle(value => value.Angle); //use the angle property as angle for the plotting

//Open, High, Low and Close
var mapper = Mappers.Financial&lt;OhlcPoint>()
                .X((value, index) => index)
                .Open(value => value.Open)
                .High(value => value.High)
                .Low(value => value.Low)
                .Close(value => value.Close);
```

You can set a mapper in several ways.

### Global mapper

This method saves the configuration at your application level, every time LiveCharts detects this type in a  *IChartValues* instance, it will use this mapper only if the SeriesCollection mapper and Series mapper are null.

```
var mapper1 = Mappers.Xy&lt;int&gt;()
  .X((value, index) =&gt; index) 
  .Y(value =&gt; value);
LiveCharts.Charting.For&lt;int&gt;(mapper1, SeriesOrientation.Horizontal); //when horizontal

var mapper2 = Mappers.Xy&lt;int&gt;()
  .X(value =&gt; value) //use the value (int) as X
  .Y((value, index) =&gt; index);
LiveCharts.Charting.For&lt;int&gt;(mapper2, SeriesOrientation.Vertical); //when vertical
```

Another sample with a custom class, the *ObservablePoint* class that only contains 2 properties, *X* and *Y*, notice this time we are using the same configuration for both, Horizontal and Vertical, without passing the second argument (Orientation).

```
var mapper = Mappers.Xy&lt;ObservablePoint&gt;()
  .X((value, index) =&gt; value.X) 
  .Y(value =&gt; value.Y);
LiveCharts.Charting.For&lt;ObservablePoint&gt;(mapper);
//that's it, now LiveCharts learned to plot the ObservablePoint class
```

If this is not clear enough you find a more detailed explanation at the <a href="https://github.com/beto-rodriguez/Live-Charts/blob/master/Core/Charting.cs#L43">source code</a>.

### SeriesCollection mapper

When you define a Series collection instance you can also pass a default configuration, this configuration overrides the global configuration and will be set only if Series configuration is null.

```
var mapper = Mappers.Xy&lt;MyClass&gt;().X(v => v.XProp).Y(v => v.YProp);
var seriesCollection = new SeriesCollection(mapper);
myChart.SeriesCollection = seriesCollection;
```

### Series mapper

Finally you can also define a mapper only for a series, this will override the globally and SeriesCollection configuration.

```
var mapper = Mappers.Xy&lt;MyClass&gt;().X(v => v.XProp).Y(v => v.YProp);
var pieSeries = new PieSeries(mapper);
```