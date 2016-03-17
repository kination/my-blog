
While working on updating old codes on Android app, one of my plan is to switch ListView to RecyclerView. It has been supported after Android 5.0.
This is a container which has better performance on displaying large dataset and scrolling up/down than ListView or GridView. It looks helpful who would need to make a list with large data, or showing lots of images.

To use it, first thing to do is to add library on gradle.

<% highlight groovy%>

dependencies {
    ...
    compile "com.android.support:recyclerview-v7:23.0.1"
}

<% endhighlights %>

Every guide I check mentioned to add this, but my code works well without it. It looks like it has been included to support UI library I added.

What on the list is showing bunch of files inside the device, and what it needs to be shown is only icon and text.

<% highlight java%>

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_sample, container, false);
        ListView listView = (ListView) view.findViewById(R.id.fragment_listView);
        listView.setAdapter(new ItemAdapter());

<% endhighlights %>

Now, I changed to RecyclerView.

<% highlight java%>

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_sample, container, false);
        RecyclerView recyclerView = (RecyclerView) view.findViewById(R.id.fragment_listView);
        LinearLayoutManager layoutManager = new LinearLayoutManager(getContext());
        layoutManager.setOrientation(LinearLayoutManager.VERTICAL);
        recyclerView.setLayoutManager(layoutManager);
        recyclerView.setAdapter(new ItemAdapter());

    }

<% endhighlights %>

This part is almost same, except the code about LinearLayoutManager.
In Android dev site, it said...
"In contrast to other adapter-backed views such as ListView or GridView, RecyclerView allows client code to provide custom layout arrangements for child views. These arrangements are controlled by the RecyclerView.LayoutManager. A LayoutManager must be provided for RecyclerView to function."

It works well after setup LayoutManager.
...


Adapter has also been changed.
Below ones are not the full code. It is just for explaining the difference between implementing adapter for ListView and RecyclerView.

<% highlight java%>

public class ItemAdapter extends BaseAdapter {

  List<Item> itemList;
  ...

  @Override
  public int getCount() {
    return itemList.size();
  }

  @Override
  public Object getItem(int position) {
    return itemList.get(position);
  }

  @Override
  public View getView(int position, View convertView, ViewGroup parent) {
    convertView = mLayoutInflater.inflate(R.layout.fragment_single_item, parent, false);
    ImageView itemIcon = (ImageView) convertView.findViewById(R.id.item_icon);
    TextView itemName = (TextView) convertView.findViewById(R.id.item_name);

    itemIcon.setImageResource(itemList.get(position).icon);
    itemName.setText(itemList.get(position).name);

    return convertView;
  }
}

<% endhighlights %>

And, this is for RecyclerView.

<% highlight java%>

public class ItemAdapter extends RecyclerView.Adapter<ItemHolder> {

  List<Item> itemList;
  ...

  @Override
  public ItemHolder onCreateViewHolder(ViewGroup parent, int viewType) {

    View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.fragment_single_item, parent, false);
    return new ItemHolder(view); //call holder to design item
  }

  @Override
  public void onBindViewHolder(ItemHolder holder, final int position) {
    Item item = itemList.get(position);
    holder.bindItem(itemList.get(position));
    holder.itemContainer.setOnClickListener(new View.OnClickListener() {
      @Override
      public void onClick(View v) {

          Logger.d(LOG_TAG, "click recyclerview item = " + itemList.get(position).getExtension());
      }
    });

    holder.itemContainer.setOnLongClickListener(new View.OnLongClickListener() {
      @Override
      public boolean onLongClick(View v) {
          Logger.d(LOG_TAG, "click recyclerview item long = " + position);
          return false;
      }
    });
  }

  @Override
  public long getItemId(int position) {
    return position;
  }

  @Override
  public int getItemCount() {
    return itemList.size();
  }
}

...

public class ItemHolder extends RecyclerView.ViewHolder {

    public View itemContainer;
    private ImageView mItemIcon;
    private TextView mItemName;
    private ItemModel mItemModel
    private ItemObj mItemObj;

    public ItemHolder(View itemView) {
        super(itemView);
        itemContainer = itemView;
        //define resources
        mItemIcon = (ImageView) itemView.findViewById(R.id.item_icon);
        mItemName = (TextView) itemView.findViewById(R.id.item_name);
    }

    //setup value for each items.
    public void bindItem(ItemModel itemModel) {
        mItemModel = itemModel;
        mItemIcon.setImageResource(mItemObj.icon);
        mItemName.setText(mItemObj.name);
    }
}

<% endhighlights %>

Now, there are some changes on override methods, and new thing name "ViewHolder".
This describes a view form and data of each items on RecycleView. RecyclerView.Adapter implementation should subclass this and add fields for its parameters. If you compare two code above about Adapter, you can find getView implementation on BaseAdapter has been divided to RecyclerView.onCreateViewHolder and RecyclerView.ViewHolder constructor.
After that, call method to bind data to view component in onBindViewHolder(...) on RecyclerView.Adapter.

The View variable(ItemHolder.itemContainer) is to handle click event on Adapter class, to give an event when item on list is selected. There are lots of way to implement select event, so this parameter is not necessary.


One more thing. After you see the list, you could find no effects are working when you touch the list. Add below on your item resource. NOT list, but item.

<% highlight xml%>

<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    ...
    android:background="@drawable/item_list_background"
    android:clickable="true"
    android:focusable="true">
    ...
<% endhighlights %>

Now converting ListView to RecyclerView is finished. If your list has few items or takes low resource, it will have no difference with previous one. It will show benefit when your list needs to handle massive data/resource in list. 
