<template>
    <div class="card">
        <div class="card-header">
            {{ $t("Comments") }}
            <div class="card-header-actions">
                <button
                    type="button"
                    class="btn btn-sm btn-primary"
                    @click="showCommentForm()"
                >
                    <i class="la la-plus"></i>
                </button>
            </div>
        </div>

        <div class="card-body">
            <div v-if="isValidated" class="alert alert-success">
                {{ $t('Your comment has been saved') }}
            </div>
            <ul>
                <li v-for="(comment, index) in commentlist" :key="comment.id">
                    {{ comment.body }} ({{
                        comment.created_at | moment("D MMM YY")
                    }})
                    <button
                        type="button"
                        class="btn btn-danger btn-sm"
                        @click="deleteComment(comment.id, index)"
                    >
                        X
                    </button>
                    <!-- <button type="button" @click="editComment(comment.id)" class="btn btn-info btn-sm">Edit</button> -->
                </li>
            </ul>
        </div>
        <div v-if="showEditField" class="card-footer">
            <div v-if="errors" class="alert alert-danger">
                {{ errors }}
            </div>

            <textarea
                id="comment"
                ref="comment"
                v-model="comment_body"
                name="comment"
                style="width: 100%;"
                rows="3"
            ></textarea>
            <div v-if="this.type == 'App\\Models\\Student'" class="form-group">
                <label for="action">{{
                    $t("This comment requires an action")
                }}</label>
                <input
                    id="action"
                    v-model="action"
                    name="action"
                    type="checkbox"
                />
            </div>
            <div class="btn-group">
                <button
                    type="button"
                    class="btn btn-default"
                    @click="showEditField = false"
                >
                    {{ $t("Cancel") }}
                </button>
                <button
                    type="button"
                    class="btn btn-primary"
                    @click="addComment()"
                >
                    {{ $t("Save") }}
                </button>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: ["comments", "id", "type", "route"],

    data() {
        return {
            comment_body: null,
            action: false,
            showEditField: false,
            errors: null,
            commentlist: this.comments,
            isValidated: false,
        };
    },

    mounted() {},

    methods: {
        showCommentForm() {
            this.showEditField = true;
            this.$nextTick(() => {
                this.$refs.comment.focus();
            })
        },

        addComment() {
            axios
                .post(this.route, {
                    body: this.comment_body,
                    commentable_id: this.id,
                    commentable_type: this.type,
                    action: this.action,
                })
                .then((response) => {
                    this.commentlist.push(response.data);
                    this.comment_body = null;
                    this.showEditField = false;
                    this.errors = null;
                    this.isValidated = true;
                    setTimeout(() => {
                        this.isValidated = false;
                    }, 3000)
                })
                .catch((e) => {
                    this.errors = e.response.data.errors.body[0];
                });
        },

        deleteComment(comment, index) {
            axios
                .delete("/comment/" + comment)
                .then((response) => {
                    this.$delete(this.commentlist, index);
                })
                .catch((e) => {
                    this.errors.push(e);
                });
        },
    },
};
</script>
