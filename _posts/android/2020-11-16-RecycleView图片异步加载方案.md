[TOC]

# 问题描述

重写RecycleView.Adapter，onBindViewHolder
``` java
    @Override
    public void onBindViewHolder(@NonNull VH holder, int position) {
        FileListItem item = listItems.get(position);
        holder.name.setText(item.getName());

        if (!isScrolling()) {
            loadBitmap(item, holder.image);
        }

        holder.itemView.setOnClickListener(new OnItemChildClickListener(1, position));//todo
    }
```

> RecycleView的Adapter就是在这个方法里，将数据和视图进行绑定的，如果在这直接加载绑定图片数据，小量数据感受不出来
> 当加载大量图片时，会造成RecycleView加载缓慢并卡顿

# 异步加载方案

直接上代码
``` java
 private class BitmapTask {
        private WeakReference<ImageView> reference; //View和BitmapTask任务相互引用，一方需要使用弱引用
        private Future future;
        private String path;

        public BitmapTask(@NonNull ImageView imageView, @NonNull String imgPath) {
            reference = new WeakReference<>(imageView);
            path = imgPath;
        }

        public boolean needAndExeCancel(String imgPath) {
            if (!imgPath.equals(path)) {
                cancle();
                return true;
            }
            return false;
        }

        public void cancle() {
            if (future != null && !future.isCancelled()) {
                future.cancel(true);
            }
        }

        public void execute() {
            future = Sync.submit(new Runnable() {
                @Override
                public void run() {
                    String key = Digest.md5(path);

                    Bitmap image = diskLruCache.get(key, (Bitmap) null);//先从磁盘中获取缓存

                    if (image == null) {
                        image = PictureManager.createImageThumbnail(path, ww, hh);//创建略缩图
                        diskLruCache.put(key, image); 
                    }

                    addBitmapToMemoryCache(path, image); //存入内存

                    ImageView imageView = reference.get();
                    if (imageView != null) {
                        imageView.setImageBitmap(image);
                    }
                }
            });
        }
    }
```

ImageView和Task绑定

``` java
    private void loadBitmap(FileListItem item, ImageView view) {
        final String path = item.getPath();
        Bitmap bitmap = getBitmapFromMemCache(path); //先从内存获取缓存

        view.setImageBitmap(bitmap);
        if (bitmap != null) {
            return;
        }

        BitmapTask task = ((BitmapTask) view.getTag());
        if (task == null || task.needAndExeCancel(path)) { //判断是否需要重新创建任务，减少重复创建任务
            task = new BitmapTask(view, path);
            task.execute();

            view.setTag(task);
        }
    }
```

# 总结

+ 上述主要是通过 Thread + Future方案实现的一个图片异步加载方案，还加入了缓存来避免重复加载耗时。
其实之前看到网上都是重写AsyncTask的方案，但发现安卓Api已经不推荐使用了，通用的一些图片异步加载框架也很完备，但依赖包有点大，看到一篇blog说基本用Thread + Future基本能解决大部分异步问题，所以就自己动手写了

+ 上述只是解决了异步加载图片问题，加载也成功了，不像之前那么卡顿，但是当快速滑动的时候，发现还是卡顿，这又是什么问题?