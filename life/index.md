# 读书、观影和旅行记录


<style>
  @import url('https://fonts.googleapis.com/css?family=Montserrat:300,400,700,800');
  * {
    box-sizing: border-box;
    margin: 0;
  }
  html, body {
    margin: 0;
    font-family: 'Montserrat', helvetica, arial, sans-serif;
    font-size: 14px;
    font-weight: 400;
  }
  .card {
    position: relative;
    display: block;
    width: 800px;
    height: 350px;
    margin: 100px auto;
    overflow: hidden;
    border-radius: 10px;
    transition: all 0.4s;
  }
  .card:hover {
    transform: scale(1.02);
    transition: all 0.4s;
  }
  .card .info_section {
    position: relative;
    width: 100%;
    height: 100%;
    background-blend-mode: multiply;
    z-index: 2;
    border-radius: 10px;
  }
  .card .info_section .header {
    position: relative;
    padding: 25px;
    height: 40%;
  }
  .card .info_section .header h1 {
    font-weight: 400;
  }
  .card .info_section .header .time {
    display: inline-block;
    margin-top: 10px;
    padding: 2px;
    border-radius: 5px;
  }
  .card .info_section .header .locandina {
    position: relative;
    float: left;
    margin-right: 20px;
    height: 120px;
  }
  .card .info_section .description {
    padding: 25px;
    height: 50%;
    color: #3e3f44;
  }
  .card .blur_back {
    position: absolute;
    top: 0;
    z-index: 1;
    height: 100%;
    right: 0;
    background-size: cover;
    border-radius: 11px;
  }
  @media screen and (min-width: 768px) {
    .header {
      width: 60%;
    }
    .description {
      width: 50%;
    }
    .info_section {
      background: linear-gradient(to right, #ffffff 50%, transparent 100%);
    }
    .blur_back {
      width: 80%;
      background-position: -100% 10% !important;
    }
  }
  @media screen and (max-width: 768px) {
    .card {
      width: 95%;
      margin: 70px auto;
      min-height: 350px;
      height: auto;
    }
    .blur_back {
      width: 100%;
      background-position: 50% 50% !important;
    }
    .header {
      width: 100%;
      margin-top: 85px;
    }
    .description {
      width: 100%;
    }
    .info_section {
      background: linear-gradient(to top, white 50%, transparent 100%);
      display: inline-grid;
    }
  }
  #bright {
    box-shadow: 0px 0px 20px -10px rgba(0, 0, 0, 0.5);
  }
  #bright:hover {
    box-shadow: 0px 0px 40px -15px rgba(0, 0, 0, 0.5);
  }
  .LostinRussia_back {
    background: url("https://s2.ax1x.com/2020/02/12/1bmBcQ.jpg");
  }
  .Basicwear_back {
    background: url("https://www.lenet.jp/magazine/wp-content/uploads/2018/01/IMG_0256_s.jpg");
  }

</style>

<div class="card" id="bright">
  <div class="info_section">
    <div class="header">
      <img class="locandina" src="https://s2.ax1x.com/2020/02/12/1bm01g.jpg"/>
      <h1>囧妈</h1>
      <span class="minutes">徐峥，袁泉<br>2020年1月25日 </span>
    </div>
    <div class="description">
      <p>
        讲述了小老板伊万缠身于商业纠纷，却意外同母亲坐上了开往俄罗斯的火车。在旅途中，他和母亲发生激烈冲突，同时还要和竞争对手斗智斗勇。为了最终抵达莫斯科，他不得不和母亲共同克服难关，并面对家庭生活中一直所逃避的问题。<br>
        <a href="https://movie.douban.com/subject/30306570/?tag=%E7%83%AD%E9%97%A8&from=gaia_video">豆瓣链接</a> 
      </p>
    </div>
  </div>
  <div class="blur_back LostinRussia_back"></div>
</div>

<div class="card" id="bright">
  <div class="info_section">
    <div class="header">
      <img class="locandina" src="https://s2.ax1x.com/2020/02/12/1bmxjH.jpg"/>
      <h1>基本穿搭</h1>
      <span class="minutes">[日]大山旬<br>20201月11日-1月14日</span>
    </div>
    <div class="description">
      <p>
        对穿衣搭配感到不耐烦，认为时尚很麻烦，穿什么都可以或者对衣服有着自己的想法但不够自信，本书就是为这样的人而准备的穿衣指南。不需要追随瞬息万变的时尚潮流，也不需要烦恼色彩搭配，只要掌握最低限度的知识和备齐常规单品，谁都能完成清爽得体的 80分搭配。<br>
        <a href="https://book.douban.com/subject/30435330/">豆瓣链接</a>
      </p>
    </div>
  </div>
  <div class="blur_back Basicwear_back"></div>
</div>

