<template>
  <div v-if="!isLoggedIn">
    <router-link :to="{ name: $routesNames.authLogin }">Sign in</router-link>
    or
    <router-link :to="{ name: $routesNames.authRegister }">sign up</router-link>
    to add comments on this article.
  </div>
  <div v-else>
    <common-errors-list :errors="errors" />
    <form class="card comment-form">
      <div class="card-block">
        <textarea
          v-model="body"
          :disabled="isLoading"
          class="form-control"
          placeholder="Write a comment..."
          rows="3"
        ></textarea>
      </div>
      <div class="card-footer">
        <img :src="userImage" class="comment-author-img" />
        <button
          class="btn btn-sm btn-primary"
          :disabled="isLoading"
          @click="addComment"
        >
          Post Comment
        </button>
      </div>
    </form>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";

import CommonErrorsList from "@/components/CommonErrorsList.vue";
import Article from "@/store/modules/Article";
import User from "@/store/modules/User";
import { isArrayOfStrings } from "@/utils/ArrayUtils";
import { notifySuccess } from "@/utils/NotificationUtils";

@Component({ components: { CommonErrorsList } })
export default class CommentAdd extends Vue {
  @Prop({ required: true }) slug!: string;

  body = "";
  errors: string[] = [];
  isLoading = false;

  get userImage(): string | null | undefined {
    return User.currentUser?.image;
  }

  get isLoggedIn(): boolean {
    return User.isLoggedIn;
  }

  async addComment(): Promise<void> {
    this.errors = [];

    this.isLoading = true;
    try {
      await Article.addComment({
        slug: this.slug,
        params: { body: this.body }
      });
      this.$emit("comment-added");
      this.body = "";
      notifySuccess("Comment is added sucessfully");
    } catch (e) {
      if (isArrayOfStrings(e)) {
        this.errors = e;
      } else {
        throw e;
      }
    } finally {
      this.isLoading = false;
    }
  }
}
</script>
