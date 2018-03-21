% MAD - Android 9: ListView / RecyclerView
% Patrick Sturm
% 21.03.2018

## Information

* Any issues with this presentation? Write a ticket or send me a pull request ;).
* Repo: [https://github.com/siyb/tw-mad-9-listview-recyclerview](https://github.com/siyb/tw-mad-9-listview-recyclerview)

# Agenda

## Agenda

* ListView
* RecyclerView

# ListView


## ListView - 1 - Resources

* Lesson: 
    * [https://developer.android.com/resources/tutorials/views/hello-listview.html](https://developer.android.com/resources/tutorials/views/hello-listview.html)
* Javadoc:
    * [https://developer.android.com/reference/android/widget/ListView.html](https://developer.android.com/reference/android/widget/ListView.html)
    * [https://developer.android.com/reference/android/widget/ArrayAdapter.html](https://developer.android.com/reference/android/widget/ArrayAdapter.html)
    * [https://developer.android.com/reference/android/widget/CursorAdapter.html](https://developer.android.com/reference/android/widget/CursorAdapter.html)
    * [https://developer.android.com/reference/android/widget/ListAdapter.html](https://developer.android.com/reference/android/widget/ListAdapter.html)


## ListView - 2 - Basics

* Now that we have learned how to access data from a database, it's time to check out a simple form of data representation, the ListView
* ListViews are an excellent way to present a data set to the user
* There are three options to include a ListView in your Activity
    * 1) using ListActivity, requires a ListView with id @id/android:list and a TextView with id @id/android:empty (will be displayed if there is no data available)
    * 2) using a normal Activity / Fragment and providing a ListView with an arbitrary id
    * 3) using a ListFragment, same requirements as in ListActivities
* ListViews use Adapters to display data, we will learn about:
    * CursorAdapters, for data acquired from a database
    * ArrayAdapters, for "static" data (or data from a webservice)

## ListView - 3 - Example: Specialized Fragment / Activity

```java
public class MyList(Activity/Fragment) 
  extends (ListActivity/ListFragment) { 
  // for Activities - Inflate before setting adapter 
  @Override 
  public void onCreate(Bundle b) { 
    setContentView(R.layout.mylistactivity); 
    setListAdapter(dummy); 
  } 
  // for Fragments - inflate before setting adapter 
  @Override 
  public View onCreateView(LayoutInflater inflater, 
    ViewGroup root, Bundle bundle){ 
    View v = inflater
      .inflate(R.layout.mylistfragment, root, false); 
    setListAdapter(dummy); 
    return v; 
  } 
```

## ListView - 4 - Example: Specialized Fragment / Activity cont.


```java
  // listitem clicks are observed using this method 
  @Override 
  public void onListItemClick(ListView listView, 
    View view, int position, long id){
    // ArrayAdapter: returns the item at 
    // the click position Adapter: 		
    // jumps to position in cursor (default) 
    // we will learn how to return an 		
    // item from this call though ;) 
    getListAdapter().getItem(position); 
  } 
}
```

## ListView - 5 - Example: Normal Activity / Fragment

```java
public class My(Activity/Fragment) 
  extends (Activity/Fragment)
  implements OnItemClickListener { 
  private MyAdapter myAdapter; 
  private ListView myListView; 
  @Override public void onCreate(Bundle b) { 
    setContentView(R.layout.myactivity); 
    myListView = findViewById(R.id.mylistview); 
    myListView.setAdapter((myAdapter = new MyAdapter(...))); 
  }
  @Override public void onItemClick(
    AdapterView<?> listView, View convertView, 
    int position, long id) { 
    myAdapter.getItem(position); 
  } 
  // for fragments, about the same, I bet you figure it out yourself ;) 
}
```

## ListView - 6 - Introduction into Adapters

* As mentioned before, we need adapters to fill ListViews
* The Adapter will receive callbacks from the ListView, telling it when to load which data set
    * All we need to do is implement these callback methods correctly and we will be fine
* First rule of ListView: Don't make assumptions about ListView
* Since memory on mobile devices is sparse, ListView tries to recycle Views as much as possible
    * A ListView will only contain a certain amount of Views (visible Views)
    * The Adapter will receive Views to recycle, meaning that they need to be fully (re)set!
    * Usually there are only a few more recycleable Views than visible Views (e.g. 10 Views are visible == total pool of 12 Views)

## ListView - 7 - Introduction into Adapters cont.

* In order to speed things up, we can use the ViewHolder pattern, which minimizes View lookups (don't worry, explanation later)
* The onItemClick and onListItemClick methods are mutually exclusive, do not add a OnItemClickListener within a ListActivity manually, it will mess things up (event consumption).
* Some views will steal focus and prevent onItemClick and onListItemClick from functioning normally (prominent example: TextView). Setting setFocusable(false) helps

## ListView - 8 - Example: CursorAdapter

```java
public class MyCursorAdapter extends CursorAdapter { 
  private DatabaseProxyMyData proxy 
    = new DatabaseProxyMyData(); 
  @Override
  public void bindView(View view, 
    Context context, Cursor cursor) { 
    View viewHolder = (ViewHolder) view.getTag(); 
    viewHolder.myText
      .setText(getItem(position).getMyString()); 
  } 
  @Override public View newView(Context context, 
    Cursor cursor, ViewGroup parent) { 
    View convertView = LayoutInflater.from(getContext()) 
      .inflate(R.layout.myitemlayout, parent, false); 
    ViewHolder h = new ViewHolder(); 

```

## ListView - 9 - Example: CursorAdapter cont.

```java
    h.myText = (TextView) convertView
      .findViewById(R.id.myText); 
    convertView.setTag(h);	
    return convertView
  }
  @Override 
  public MyData getItem(int postition) {
    Cursor c = getCursor(); 
    c.moveToPosition(position); 
    proxy.setDataObject(c); 
    return proxy.getDataObject(); 
  }
  private static final class ViewHolder { 
    private TextView myText; 
  } 
}
```

## ListView - 10 - Example: ArrayAdapter

```java
public class MyArrayAdapter extends ArrayAdapter<MyData> {
  public MyArrayAdapter(Context context, int resource, 
    int textViewResourceId, List<MyData> objects) { 
    super(context, resource, textViewResourceId, objects); 
  } 
  @Override 
  public View getView(int position, 
    View convertView, ViewGroup parent) { 
    if (convertView == null) { 
      // inflate + viewholder 
    } 
    MyData item = getItem(position); 
    // bind data to view 
  } 
} 
```

## ListView - 11 - CursorAdapter vs. ArrayAdapter

* Biggest difference: data source
    * Please do not attempt to use ArrayAdapters with databases
* Implementation difference:
    * ArrayAdapters are generic
    * ArrayAdapters use getView(...)
    * CursorAdapters use newView(...) and bindView(...)
        * You can use getView(...) as well (In my opinon that's the more consistent way of doing things)
    * CursorAdapter.getItem(...) does not work as expected
        * Will move to position in Cursor and return Cursor


# RecyclerView

## RecyclerView - 1 - Resources

* Lesson:
    * [https://developer.android.com/training/material/lists-cards.html](https://developer.android.com/training/material/lists-cards.html)
* Javadoc:
    * [https://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html)
    * [https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html)
    * [https://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter.html](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter.html)
    * [https://developer.android.com/reference/android/support/v7/widget/RecyclerView.ViewHolder.html](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.ViewHolder.html)


## RecyclerView - 2 - Introduction

* „new“ API – can be used instead of ListView / GridView
* Part of the support library
* Implements some new concepts
    * RecyclerView is not a ListView
    * The ViewHolder pattern is an intricate part of the RecyclerView
    * Has its own implementation of Adapter
        * No more ArrayAdapter / CursorAdapter, data source needs to be managed by the user
        * ViewHolder based
    * No more specialized Fragments / Activities!
* Allows using LayoutManager – much more flexibility, changing behaviour (grid, horizontal, vertical, etc) by supplying fitting LayoutManagers

## RecyclerView - 3 - Example: Adapter

```java
public class MyAdapter 
  extends RecyclerView.Adapter<MyAdapter.ViewHolder> {
  private List<MyData> objects = new ArrayList<>();
  @Override
  public ViewHolder onCreateViewHolder(ViewGroup parent,
    int viewType) {
    View v = LayoutInflater.from(parent.getContext())
      .inflate(R.layout.,parent, false);
    return new ViewHolder(v);
  }
  @Override
  public void onBindViewHolder(ViewHolder holder,
    int position) {
    MyData item = objects.get(position);
    holder.myImageView
  }
```

## RecyclerView - 4 - Example: Adapter cont.

```java
  @Override
  public int getItemCount() {
    return objects.size();
  }
  protected static final class ViewHolder
    extends RecyclerView.ViewHolder {
    private ImageView myImageView;
    public ViewHolder(View itemLayoutView) {
      super(itemLayoutView);
      this.myImageView = (ImageView) 
        itemLayoutView.findViewById(R.id.myImageVIew);
    }
  }
} 
```

## RecyclerView - 5 - Example: Usage

```java
public class MyFragment extends Fragment {
  private MyAdapter myAdapter;
  private RecyclerView myRecyclerView;
  public View onCreateView(LayoutInflater inflater, 
    ViewGroup container, Bundle savedInstanceState) {
    View v = inflater
      .inflate(R.layout.myFragment, container, false);
    myRecyclerView = (RecyclerView) v
      .findViewById(R.id. MyRecyclerView);
    LinearLayoutManager layoutManager = 
      new LinearLayoutManager(getContext(),
        LinearLayoutManager.HORIZONTAL,
        false);
    myRecyclerView.setLayoutManager(layoutManager);
    myRecyclerView.setAdapter(new MyAdapter(...));
    return v;
  }
}
```

# Any Questions?
