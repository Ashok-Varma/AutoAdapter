# AutoAdapter
This Repository simplifies working with RecyclerView Adapter

## Gradle:
Add it in your root build.gradle at the end of repositories:
```Groovy
	allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}
```
Add dependency to app gradle:
```Groovy
compile 'com.github.zuluft:autoadapter:v1.1'
```

## Usage:

### Step 1:
create layout xml file:
```
item_footballer.xml
```
```xml
<android.support.v7.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:cardUseCompatPadding="true">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:orientation="horizontal">

        <TextView
            android:id="@+id/tvName"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:layout_marginLeft="10dp"
            android:layout_weight="1"
            android:lines="1"
            android:maxLines="1"
            android:text="Zuluft"
            android:textSize="20sp" />

        <ImageView
            android:id="@+id/ivDelete"
            android:layout_width="30dp"
            android:layout_height="30dp"
            android:layout_gravity="center"
            android:layout_marginLeft="10dp"
            android:layout_marginRight="10dp"
            android:src="@drawable/ic_delete" />

    </LinearLayout>

</android.support.v7.widget.CardView>
```

### Step 2 (Optional):
create model, which you want to be drawn on above created layout:
```Java
public class FootballerModel {
    private final String name;
    private final int number;
    private final String team;


    public FootballerModel(String name, int number, String team) {
        this.name = name;
        this.number = number;
        this.team = team;
    }

    public int getNumber() {
        return number;
    }

    public String getTeam() {
        return team;
    }

    public String getName() {
        return name;
    }
}
```
### Step 3:
create descendant of ```AutoViewHolder``` with ```ViewHolder``` annotation:

```Java
    @ViewHolder(R.layout.item_footballer)
    public class FootballerViewHolder extends AutoViewHolder {
        private TextView tvName;

        public FootballerViewHolder(View itemView) {
            super(itemView);
            tvName = (TextView) findViewById(R.id.tvName);
        }
    }
```
by this line 
```Java
 @ViewHolder(R.layout.item_footballer)
``` 
we are telling ```AutoAdapter``` that we want to inflate ```FootballerViewHolder```'s ```itemView``` with ```item_footballer``` layout. 

### Step 4:
create descedent of ```Renderable```  with ```Renderer``` annotation:
(in this example ```FootballerViewHolder``` is the inner static class of the ```FootballerRenderer```)

```Java
@Renderer(FootballerRenderer.FootballerViewHolder.class)
public class FootballerRenderer extends Renderable<FootballerRenderer.FootballerViewHolder> {

    public final FootballerModel mFootballerModel;

    public String getUsername() {
        return mFootballerModel.getName();
    }

    public FootballerRenderer(FootballerModel footballerModel) {
        this.mFootballerModel = footballerModel;
    }

    @Override
    public void apply(FootballerViewHolder viewHolder) {
        viewHolder.tvName.setText(mFootballerModel.getName());
    }

    @ViewHolder(R.layout.item_footballer)
    public static class FootballerViewHolder extends AutoViewHolder {
        private TextView tvName;

        public FootballerViewHolder(View itemView) {
            super(itemView);
            tvName = (TextView) findViewById(R.id.tvName);
        }
    }
}
```

by this line 
```Java
 @Renderer(FootballerRenderer.FootballerViewHolder.class)
```
we are telling ```AutoAdapter``` that we want to use instances of ```FootballerViewHolder``` for ```FootballerRenderer``` items.

```Renderable``` Abtsract class has only one abstract method ```apply``` with generic argument type. In our example, this type is ```FootballerViewHolder```

### Step 5:

create an ```Activity```, with the very simple ```RecyclerView``` in it's content view:
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/rvList"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:scrollbars="none" />
</RelativeLayout>
```

create ```FootballerModel``` objects:
```Java
public static FootballerModel[] getUsers() {
        return new FootballerModel[]{
                new FootballerModel("Leo Messi", 10, "Barcelona"),
                new FootballerModel("Andres Iniesta", 8, "Barcelona"),
                new FootballerModel("Neymar Jr.", 11, "Barcelona"),
                new FootballerModel("Ibrahimovic", 9, "Man. United"),
                new FootballerModel("Hazard", 10, "Chelsea"),
                new FootballerModel("Rooney", 10, "Man. United"),
                new FootballerModel("Ozil", 10, "Arsenal"),
                new FootballerModel("Cech", 1, "Chelsea"),
                new FootballerModel("Luis Suarez", 9, "Barcelona"),
        };
    }
```
convert them to ```FootballerRenderer```s:
```Java
public FootballerRenderer[] convertToRenderer(FootballerModel[] footballerModels) {
        FootballerRenderer[] footballerRenderers = new FootballerRenderer[footballerModels.length];
        for (int i = 0; i < footballerModels.length; i++) {
            footballerRenderers[i] = new FootballerRenderer(footballerModels[i]);
        }
        return footballerRenderers;
    }
```
create the ```AutoAdapter``` instance, add ```FootballerRenderer``` objects and set it to ```RecyclerView```
```Java
mRecyclerView = (RecyclerView) findViewById(R.id.rvList);
mRecyclerView.setLayoutManager(new LinearLayoutManager(this));
AutoAdapter autoAdapter = new AutoAdapter();
autoAdapter.addAll(convertToRenderer(getUsers()));
mRecyclerView.setAdapter(autoAdapter);
```

if you want to listen item clicks:
```Java
autoAdapter.bindListener(FootballerRenderer.class,
                itemInfo -> toastName(itemInfo.object.getUsername()));
```
or if you want to listen clicks of any ```itemView```s child views, for example view with ```ivDelete``` id:
```Java
autoAdapter.bindListener(FootballerRenderer.class, R.id.ivDelete,
                itemInfo -> autoAdapter.remove(itemInfo.position));
```
this is ```ItemInfo``` class:
```Java
public class ItemInfo<T extends IRenderable, V extends AutoViewHolder> {
    public final int position;
    public final T object;
    public final V viewHolder;

    public ItemInfo(int position, T object, V viewHolder) {
        this.position = position;
        this.object = object;
        this.viewHolder = viewHolder;
    }
}
```
in our example, ```T```=```FootballerRenderer``` and ```V```=```FootballerViewHolder```

# Mmm... If I want to have heterogeneous items and layouts inside ```RecyclerView``` ?
AutoAdapter's working perfectly with heterogeneous items.
You can add any descedent of ```Renderable``` to ```AutoAdapter```, for example it's not a problem to write the following:
```Java
AutoAdapter autoAdapter=new AutoAdapter();
autoAdapter.add(new FootballerRenderer());
autoAdapter.add(new BasketballerRenderer());
autoAdapter.add(new BoxerRenderer());
....
```
They all will draw their own layout. They can use the same ```ViewHolder``` too.


# This is f***ing awsame ! anything else ?
```Auto Adapter``` is using ```SortedList``` and in every descedent of ```Renderable``` we can override these methods. 
here are their default implementations:
```Java
public abstract class Renderable<T extends AutoViewHolder> implements IRenderable<T> {

    @Override
    public int compareTo(IRenderable item) {
        return 0;
    }

    @Override
    public boolean areContentsTheSame(IRenderable item) {
        return false;
    }

    @Override
    public boolean areItemsTheSame(IRenderable item) {
        return false;
    }
}
```
For example, if we want ```FootballRenderers``` to be sorted by ```FootballerModel```s ```number``` property, we can override ```compareTo``` method:
```Java
    @Override
    public int compareTo(IRenderable item) {
        FootballerModel otherFootballer = ((FootballerRenderer) item).mFootballerModel;
        return Integer.compare(mFootballerModel.getNumber(), otherFootballer.getNumber());
    }
```

