<template>
		<div>
		<base-header type="sb-info" class="pb-6 pb-8 pt-5 pt-md-8 background" style="height: calc(100vh + 6rem);" :style="{'background-image':`url(${backgroundImg})`}">
        </base-header>
		<div class="container-fluid mt--7">
			<div class="row" style="margin-bottom: 10px;">
					<div class="col-xl-8 mb-5 mb-xl-0">
						<lRanking />
					</div>
				
				<div class="col-xl-4 md-5">
					<lTags />
				</div>
			</div>
			<div class="row">
				<div class="col justify-content-center">
					<Waterfall v-if="rankimg.length>0" ref="waterfall" imageType="medium" :images="rankimg" :cardWidth="cardWidth"/>
					<infinite-loading @infinite="infiniteHandler" :identifier="waterfallIdentifier" spinner="spiral"></infinite-loading>
				</div>
			</div>
		</div>
    </div>
</template>
<script>
import lRanking from './components/lRanking';
import lTags from './components/lTags';
import CONFIG from '@/config.json'
import Waterfall from './components/Waterfall';
import storage from 'good-storage'

export default {
    components: {
		lRanking,
		lTags,
		Waterfall
    },
    data() {
      return {
		rankimg: this.$store.state.mainpage.rankimg,
		waterfallIdentifier: Math.round(Math.random() * 100),
		cardWidth:260,
		isR18: storage.get("r18") == true ? true : storage.set("r18", false),
		mainPageSanity: storage.get("mainPageSanity") == false ? false : storage.set("mainPageSanity", true),
      };
    },
	watch:{
	},
	computed:{
		backgroundImg(){
			return CONFIG.RAND_IMG
		}
	},
    methods: {
		infiniteHandler($state) {
			if(this.rankimg.length>0){
				$state.loaded();
				$state.complete();
				return;
			}
			this.axios
				.get(CONFIG.API_HOST, {
					params: {
						type: "illust_recommended",
						}
					})
				.then((response, state) => {
					if (!response.data.illusts || response.data.illusts.length==0) {
						$state.complete();
						return;
					}
					
					this.rankimg = this.mainPageSanity?(response.data.illusts).filter(function(o){
						return o.sanity_level < 3
					}):(response.data.illusts)
					this.$store.commit("mainpage/setrankimg", this.rankimg)
					$state.loaded();
					$state.complete();
				});
		}
    },
    mounted() {
		window.scrollTo(0, this.$store.state.scroll.top)
		
			var dialog = storage.get("dialog","")
			this.axios
				.get("getyou.txt")
				.then((response, state) => {
					if(response.data!=dialog){
						this.$notify({
							type: 'warning',
							title: response.data,
							timeout:null,
							closeOnClick:false
						})
						storage.set("dialog",response.data)
					}
				});
		/*
		this.axios
			.get("https://www.rrll.cc/tuceng/ecy.php?return=json")
			.then((response, state) => {
				if (response.data.code!=200) {
					return;
				}
				this.backgroundImg = response.data.acgurl
			});
			*/
	}
  };
</script>
<style></style>
