<template>
  <footer class="footer">
    <div class="footer-container">
      <div class="footer-content">
        <div class="footer-section">
          <h3>青岚</h3>
          <p>寻青云涧，栖山水间</p>
          <p>一个分享美好事物的地方</p>
        </div>
        
        <div class="footer-section">
          <h4>快速链接</h4>
          <ul>
            <li><router-link to="/">首页</router-link></li>
            <li><router-link to="/blog">个人笔记</router-link></li>
            <li><router-link to="/recommendations">推荐</router-link></li>
            <li><router-link to="/news">资讯</router-link></li>
          </ul>
        </div>
        
        <div class="footer-section">
          <h4>联系方式</h4>
          <p>邮箱: contact@qinglan.com</p>
          <p>微信: qinglan_share</p>
        </div>
        
        <div class="footer-section">
          <h4>关注我们</h4>
          <div class="social-links">
            <a href="#" class="social-link">微博</a>
            <a href="#" class="social-link">微信</a>
            <a href="#" class="social-link">GitHub</a>
          </div>
        </div>
      </div>
      
      <div class="footer-bottom">
        <p>&copy; 2024 青岚. All rights reserved.</p>
        <p>Built with Vue.js</p>
      </div>
    </div>
  </footer>
</template>

<script>
export default {
  name: 'FooterComponent'
}
</script>

<style scoped>
.footer {
  background: linear-gradient(135deg, #2c3e50 0%, #34495e 100%);
  color: white;
  margin-top: auto;
}

.footer-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
}

.footer-content {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 2rem;
  margin-bottom: 2rem;
}

.footer-section h3,
.footer-section h4 {
  margin-bottom: 1rem;
  color: #ecf0f1;
}

.footer-section p,
.footer-section li {
  margin-bottom: 0.5rem;
  color: #bdc3c7;
}

.footer-section ul {
  list-style: none;
  padding: 0;
}

.footer-section a {
  color: #bdc3c7;
  text-decoration: none;
  transition: color 0.3s ease;
}

.footer-section a:hover {
  color: #3498db;
}

.social-links {
  display: flex;
  gap: 1rem;
}

.social-link {
  padding: 0.5rem 1rem;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 20px;
  transition: all 0.3s ease;
}

.social-link:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateY(-2px);
}

.footer-bottom {
  text-align: center;
  padding-top: 2rem;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  color: #95a5a6;
}

@media (max-width: 768px) {
  .footer-container {
    padding: 1rem;
  }
  
  .footer-content {
    grid-template-columns: 1fr;
    gap: 1rem;
  }
  
  .social-links {
    justify-content: center;
  }
}
</style>
