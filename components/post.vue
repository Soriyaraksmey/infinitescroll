<script setup>
import { useInfiniteQuery } from "@tanstack/vue-query";
import { useInfiniteScroll } from "@vueuse/core";

const fetchPost = async ({ pageParam = 0 }) => {
  const res = await fetch(`https://dummyjson.com/posts?limit=10&skip=${pageParam}&select=title`);
  return res.json();
};

const { data, isPending, isSuccess, fetchNextPage, isFetchingNextPage, hasNextPage } = useInfiniteQuery({
  queryKey: ["posts"],
  queryFn: fetchPost,
  getNextPageParam: (lastPage, pages) => {
    return lastPage?.limit === 0 ? undefined : lastPage.skip + 10;
  },
});

const el = ref(null);

useInfiniteScroll(
  el,
  async () => {
    //loadmore
    if (hasNextPage.value) {
      await fetchNextPage();
    }
  },
  { distance: 10 }
);
</script>

<template>
  <span v-if="isPending">Loading...</span>
  <div
    v-if="isSuccess"
    class="max-h-[40rem] w-[40rem] p-4 bg-slate-200 flex items-center justify-center overflow-y-scroll shadow-2xl rounded-sm"
    ref="el">
    <span v-for="(item, index) in data.pages" :key="index">
      <div class="p-4 my-2 shadow-md rounded-md bg-white" v-for="(post, index) in item.posts" :key="index">
        {{ post.title }}
      </div>
      <div class="text-center my-2" v-if="isFetchingNextPage">Loading more post...</div>
    </span>
    <div class="text-center my-2" v-if="!hasNextPage">No more data</div>
  </div>
</template>
