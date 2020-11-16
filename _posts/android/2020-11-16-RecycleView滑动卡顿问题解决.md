``` java
        //方式一 通过Detector实现
//        GestureDetector gestureDetector = new GestureDetector(MyApp.app,
//                new GestureDetector.SimpleOnGestureListener() {
//
//                    /**
//                     * @param e1 down事件
//                     * @param e2 当前move事件
//                     * @param velocityX 每秒x轴移动像素
//                     * @param velocityY 每秒y轴移动像素
//                     * @return
//                     */
//                    @Override
//                    public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
//                        if (Math.abs(velocityY) > 8000) {
//                            fileListAdapter.setScrolling(true);
//                        }
//
//                        return false;
//                    }
//                });
//
//        recyclerView.setOnTouchListener(new View.OnTouchListener() {
//            @Override
//            public boolean onTouch(View v, MotionEvent event) {
//                gestureDetector.onTouchEvent(event);
//                return false;
//            }
//        });

        //方式2 RecycleView的内部惯性监听
//        recyclerView.setOnFlingListener(new RecyclerView.OnFlingListener() {
//            @Override
//            public boolean onFling(int velocityX, int velocityY) {
//                if (velocityY >= 1000) {
//                    Logger.e("开始快滑动了");
//                    fileListAdapter.setScrolling(true);
//                }
//
//                return false;
//            }
//        });
```

``` java

        recyclerView.addOnScrollListener(new RecyclerView.OnScrollListener() {

            private boolean scrolled;

            @Override
            public void onScrollStateChanged(@NonNull RecyclerView recyclerView, int newState) {
                switch (newState) {
                    case RecyclerView.SCROLL_STATE_IDLE:// 静止
                        if (fileListAdapter.isScrolling() && scrolled) { //滑动过而且是快滑动
                            fileListAdapter.setScrolling(false);

                            Logger.e("开始加载数据");

                            fileListAdapter.notifyDataSetChanged();
                        }
                        scrolled = false;
                        break;
                    default:
                        fileListAdapter.setScrolling(true);
                        break;
                }

            }

            @Override
            public void onScrolled(@NonNull RecyclerView recyclerView, int dx, int dy) {
                if (dy != 0) {
                    scrolled = true;
                }
            }
        });
```