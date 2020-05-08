# DouYin
抖音主界面简单实现。
这里简单实现抖音主界面如下（点击查看GitHub上的Demo）：

 



抖音主界面视频这里采用RecyclerView来实现视频滚动，我们知道RecyclerView是要靠LayoutManager来管理的，我们自己来简单实现一个这个功能，重写LayoutManager。

1：自定义LayoutManager管理类如下：

package com.example.administrator.douyin;
 
import android.content.Context;
import android.support.annotation.NonNull;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.PagerSnapHelper;
import android.support.v7.widget.RecyclerView;
import android.view.View;
 
/**
 * 自定义LayoutManager来实现RecyclerView的管理
 *
 */
public class MyLayoutManager  extends LinearLayoutManager implements RecyclerView.OnChildAttachStateChangeListener {
    private int mDrift;//位移，用来判断移动方向
 
    private PagerSnapHelper mPagerSnapHelper;
    private OnViewPagerListener mOnViewPagerListener;
    public MyLayoutManager(Context context) {
        super(context);
    }
 
    public MyLayoutManager(Context context, int orientation, boolean reverseLayout) {
        super(context, orientation, reverseLayout);
        mPagerSnapHelper = new PagerSnapHelper();
    }
 
    @Override
    public void onAttachedToWindow(RecyclerView view) {
 
        view.addOnChildAttachStateChangeListener(this);
        mPagerSnapHelper.attachToRecyclerView(view);
        super.onAttachedToWindow(view);
    }
//当Item添加进来了  调用这个方法
 
//
    @Override
    public void onChildViewAttachedToWindow(@NonNull View view) {
//        播放视频操作 即将要播放的是上一个视频 还是下一个视频
        if (mDrift > 0) {
//            向上
            if (mOnViewPagerListener != null) {
                mOnViewPagerListener.onPageSelected(getPosition(view), true);
            }
 
        }else {
            if (mOnViewPagerListener != null) {
                mOnViewPagerListener.onPageSelected(getPosition(view), false);
            }
        }
    }
 
    public void setOnViewPagerListener(OnViewPagerListener mOnViewPagerListener) {
        this.mOnViewPagerListener = mOnViewPagerListener;
    }
 
    @Override
    public void onScrollStateChanged(int state) {
        switch (state) {
            case RecyclerView.SCROLL_STATE_IDLE:
               View view= mPagerSnapHelper.findSnapView(this);
                int position = getPosition(view);
                if (mOnViewPagerListener != null)
                {
                    mOnViewPagerListener.onPageSelected(position, position == getItemCount() - 1);
 
                }
//                postion  ---回调 ----》播放
 
 
                break;
        }
        super.onScrollStateChanged(state);
    }
 
    @Override
    public void onChildViewDetachedFromWindow(@NonNull View view) {
//暂停播放操作
        if (mDrift >= 0){
            if (mOnViewPagerListener != null) mOnViewPagerListener.onPageRelease(true,getPosition(view));
        }else {
            if (mOnViewPagerListener != null) mOnViewPagerListener.onPageRelease(false,getPosition(view));
        }
    }
 
 
    @Override
    public int scrollVerticallyBy(int dy, RecyclerView.Recycler recycler, RecyclerView.State state) {
        this.mDrift = dy;
        return super.scrollVerticallyBy(dy, recycler, state);
    }
 
    @Override
    public boolean canScrollVertically() {
        return true;
    }
}
2：主界面代码如下所示：

/**
 * 进入抖音首页
 * create by andyYuan317 on 2020/05/01
 */
public class MainActivity extends AppCompatActivity   {
    private static final String TAG = "test";
    private RecyclerView mRecyclerView;
    private MyAdapter mAdapter;
    MyLayoutManager myLayoutManager;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initView();
        initListener();
    }
    private void initView() {
        mRecyclerView = findViewById(R.id.recycler);
        myLayoutManager = new MyLayoutManager(this, OrientationHelper.VERTICAL,false);
 
        mAdapter = new MyAdapter();
        mRecyclerView.setLayoutManager(myLayoutManager);
        mRecyclerView.setAdapter(mAdapter);
    }
 
    private void initListener(){
        myLayoutManager.setOnViewPagerListener(new OnViewPagerListener() {
            @Override
            public void onInitComplete() {
 
            }
 
            @Override
            public void onPageRelease(boolean isNext, int position) {
                Log.e(TAG,"释放位置:"+position +" 下一页:"+isNext);
                int index = 0;
                if (isNext){
                    index = 0;
                }else {
                    index = 1;
                }
                releaseVideo(index);
            }
 
            @Override
            public void onPageSelected(int position, boolean isNext) {
                Log.e(TAG,"释放位置:"+position +" 下一页:"+isNext);
 
                int index = 0;
                if (isNext){
                    index = 0;
                }else {
                    index = 1;
                }
                playVideo(index);
            }
        });
    }
 
    class MyAdapter extends RecyclerView.Adapter<MyAdapter.ViewHolder>{
        private int[] imgs = {R.mipmap.img_video_1,R.mipmap.img_video_2};
        private int[] videos = {R.raw.video_1,R.raw.video_2};
        public MyAdapter(){
        }
        @Override
        public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
            View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_view_pager,parent,false);
            return new ViewHolder(view);
        }
 
        @Override
        public void onBindViewHolder(ViewHolder holder, int position) {
            holder.img_thumb.setImageResource(imgs[position%2]);
            holder.videoView.setVideoURI(Uri.parse("android.resource://"+getPackageName()+"/"+ videos[position%2]));
        }
 
        @Override
        public int getItemCount() {
            return 20;
        }
 
        public class ViewHolder extends RecyclerView.ViewHolder{
            ImageView img_thumb;
            VideoView videoView;
            ImageView img_play;
            RelativeLayout rootView;
            public ViewHolder(View itemView) {
                super(itemView);
                img_thumb = itemView.findViewById(R.id.img_thumb);
                videoView = itemView.findViewById(R.id.video_view);
                img_play = itemView.findViewById(R.id.img_play);
                rootView = itemView.findViewById(R.id.root_view);
            }
        }
    }
 
    private void releaseVideo(int index){
        View itemView = mRecyclerView.getChildAt(index);
        final VideoView videoView = itemView.findViewById(R.id.video_view);
        final ImageView imgThumb = itemView.findViewById(R.id.img_thumb);
        final ImageView imgPlay = itemView.findViewById(R.id.img_play);
        videoView.stopPlayback();
        imgThumb.animate().alpha(1).start();
        imgPlay.animate().alpha(0f).start();
    }
 
 
    @TargetApi(Build.VERSION_CODES.JELLY_BEAN_MR1)
    private void playVideo(int position) {
        View itemView = mRecyclerView.getChildAt(0);
        final VideoView videoView = itemView.findViewById(R.id.video_view);
        final ImageView imgPlay = itemView.findViewById(R.id.img_play);
        final ImageView imgThumb = itemView.findViewById(R.id.img_thumb);
        final RelativeLayout rootView = itemView.findViewById(R.id.root_view);
        final MediaPlayer[] mediaPlayer = new MediaPlayer[1];
        videoView.start();
        videoView.setOnInfoListener(new MediaPlayer.OnInfoListener() {
            @Override
            public boolean onInfo(MediaPlayer mp, int what, int extra) {
                mediaPlayer[0] = mp;
                mp.setLooping(true);
                imgThumb.animate().alpha(0).setDuration(200).start();
                return false;
            }
        });
        videoView.setOnPreparedListener(new MediaPlayer.OnPreparedListener() {
            @Override
            public void onPrepared(MediaPlayer mp) {
 
            }
        });
 
 
        imgPlay.setOnClickListener(new View.OnClickListener() {
            boolean isPlaying = true;
            @Override
            public void onClick(View v) {
                if (videoView.isPlaying()){
                    imgPlay.animate().alpha(1f).start();
                    videoView.pause();
                    isPlaying = false;
                }else {
                    imgPlay.animate().alpha(0f).start();
                    videoView.start();
                    isPlaying = true;
                }
            }
        });
    }
 

————————————————
版权声明：本文为CSDN博主「AndyYuan317」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_42618969/article/details/105953200
