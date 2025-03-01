#include <SFML/Graphics.hpp>
#include <SFML/Audio.hpp>
#include <vector>
#include <memory>

class GameObject {
public:
    virtual void update(float dt) = 0;
    virtual void render(sf::RenderWindow& window) = 0;
    virtual sf::FloatRect getBounds() const = 0;
    virtual ~GameObject() {}
};

class Bullet : public GameObject {
private:
    sf::RectangleShape shape;
    float speed;
public:
    Bullet(float x, float y) {
        shape.setSize(sf::Vector2f(5, 15));
        shape.setFillColor(sf::Color::Red);
        shape.setPosition(x, y);
        speed = 300.0f;
    }
    void update(float dt) override {
        shape.move(0, -speed * dt);
    }
    void render(sf::RenderWindow& window) override {
        window.draw(shape);
    }
    sf::FloatRect getBounds() const override {
        return shape.getGlobalBounds();
    }
};

class Player : public GameObject {
private:
    sf::Sprite sprite;
    sf::Texture texture;
    float speed;
    sf::SoundBuffer shootBuffer;
    sf::Sound shootSound;
public:
    std::vector<std::unique_ptr<Bullet>> bullets;

    Player() {
        texture.loadFromFile("assets/player.png");
        sprite.setTexture(texture);
        sprite.setPosition(400, 500);
        speed = 200.0f;

        shootBuffer.loadFromFile("assets/shoot.wav");
        shootSound.setBuffer(shootBuffer);
    }
    void update(float dt) override {
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Left)) sprite.move(-speed * dt, 0);
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Right)) sprite.move(speed * dt, 0);
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Space)) {
            bullets.push_back(std::make_unique<Bullet>(sprite.getPosition().x + 20, sprite.getPosition().y));
            shootSound.play();
        }
        for (auto& bullet : bullets) bullet->update(dt);
    }
    void render(sf::RenderWindow& window) override {
        window.draw(sprite);
        for (auto& bullet : bullets) bullet->render(window);
    }
    sf::FloatRect getBounds() const override {
        return sprite.getGlobalBounds();
    }
};

class Enemy : public GameObject {
private:
    sf::Sprite sprite;
    sf::Texture texture;
    float speed;
public:
    Enemy(float x, float y) {
        texture.loadFromFile("assets/enemy.png");
        sprite.setTexture(texture);
        sprite.setPosition(x, y);
        speed = 100.0f;
    }
    void update(float dt) override {
        sprite.move(0, speed * dt);
    }
    void render(sf::RenderWindow& window) override {
        window.draw(sprite);
    }
    sf::FloatRect getBounds() const override {
        return sprite.getGlobalBounds();
    }
};

class Meteor : public GameObject {
private:
    sf::Sprite sprite;
    sf::Texture texture;
    float speed;
public:
    Meteor(float x, float y) {
        texture.loadFromFile("assets/meteor.png");
        sprite.setTexture(texture);
        sprite.setPosition(x, y);
        speed = 150.0f;
    }
    void update(float dt) override {
        sprite.move(0, speed * dt);
    }
    void render(sf::RenderWindow& window) override {
        window.draw(sprite);
    }
    sf::FloatRect getBounds() const override {
        return sprite.getGlobalBounds();
    }
};

class Game {
private:
    sf::RenderWindow window;
    Player player;
    std::vector<std::unique_ptr<Enemy>> enemies;
    std::vector<std::unique_ptr<Meteor>> meteors;
    sf::SoundBuffer explosionBuffer;
    sf::Sound explosionSound;
    sf::Texture backgroundTexture;
    sf::Sprite background;
public:
    Game() : window(sf::VideoMode(800, 600), "Shooter Espacial") {
        backgroundTexture.loadFromFile("assets/background.png");
        background.setTexture(backgroundTexture);
        
        enemies.push_back(std::make_unique<Enemy>(100, 50));
        enemies.push_back(std::make_unique<Enemy>(300, 50));
        enemies.push_back(std::make_unique<Enemy>(500, 50));
        
        meteors.push_back(std::make_unique<Meteor>(200, 0));
        meteors.push_back(std::make_unique<Meteor>(400, -50));
        
        explosionBuffer.loadFromFile("assets/explosion.wav");
        explosionSound.setBuffer(explosionBuffer);
    }
    void run() {
        sf::Clock clock;
        while (window.isOpen()) {
            sf::Time dt = clock.restart();
            handleEvents();
            update(dt.asSeconds());
            render();
        }
    }
    void handleEvents() {
        sf::Event event;
        while (window.pollEvent(event)) {
            if (event.type == sf::Event::Closed)
                window.close();
        }
    }
    void update(float dt) {
        player.update(dt);
        for (auto& enemy : enemies) enemy->update(dt);
        for (auto& meteor : meteors) meteor->update(dt);
        checkCollisions();
    }
    void checkCollisions() {
        for (auto it = player.bullets.begin(); it != player.bullets.end();) {
            bool hit = false;
            for (auto et = enemies.begin(); et != enemies.end();) {
                if ((*it)->getBounds().intersects((*et)->getBounds())) {
                    explosionSound.play();
                    it = player.bullets.erase(it);
                    et = enemies.erase(et);
                    hit = true;
                    break;
                } else {
                    ++et;
                }
            }
            if (!hit) {
                ++it;
            }
        }
    }
    void render() {
        window.clear();
        window.draw(background);
        player.render(window);
        for (auto& enemy : enemies) enemy->render(window);
        for (auto& meteor : meteors) meteor->render(window);
        window.display();
    }
};

int main() {
    Game game;
    game.run();
    return 0;
}
