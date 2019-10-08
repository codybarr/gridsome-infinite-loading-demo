<template>
	<Layout :show-logo="false">
		<!-- Author intro -->
		<Author :show-title="true" />

		<!-- List posts -->
		<div class="posts">
			<transition-group name="fade">
				<PostCard
					v-for="{ node } of loadedPosts"
					:key="node.id"
					:post="node"
				/>
			</transition-group>
			<ClientOnly>
				<infinite-loading @infinite="infiniteHandler" spinner="spiral">
					<div slot="no-more" class="mt-2">
						You've scrolled through all the posts ;)
					</div>
					<div slot="no-results" class="mt-2">
						Sorry, no posts yet :(
					</div>
				</infinite-loading>
			</ClientOnly>
		</div>
	</Layout>
</template>

<page-query>
query Blog($page: Int) {
	posts: allPost(perPage: 10, page: $page) @paginate {
		pageInfo {
			totalPages
			currentPage
		}
		edges {
			node {
				id
				title
				date (format: "D. MMMM YYYY")
				timeToRead
				description
				cover_image (width: 770, height: 380, blur: 10)
				path
				tags {
					id
					title
					path
				}
			}
		}
	}
}
</page-query>

<script>
import Author from '~/components/Author.vue'
import PostCard from '~/components/PostCard.vue'

export default {
	metaInfo: {
		title: 'Hello, world!'
	},
	components: {
		Author,
		PostCard
	},
	data() {
		return {
			loadedPosts: [],
			currentPage: 1
		}
	},
	created() {
		this.loadedPosts.push(...this.$page.posts.edges)
	},
	methods: {
		async infiniteHandler($state) {
			if (this.currentPage + 1 > this.$page.posts.pageInfo.totalPages) {
				$state.complete()
			} else {
				const { data } = await this.$fetch(`/${this.currentPage + 1}`)
				if (data.posts.edges.length) {
					this.currentPage = data.posts.pageInfo.currentPage
					this.loadedPosts.push(...data.posts.edges)
					$state.loaded()
				} else {
					$state.complete()
				}
			}
		}
	}
}
</script>
