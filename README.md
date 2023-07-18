# unity
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Platform : MonoBehaviour
{
    [SerializeField] private float _moveSpeed;

    private GameObject _player;
    private int _moveDirection;
    private bool _hasToMove = true;

    private void Awake()
    {
        _player = GameObject.FindGameObjectWithTag("Player");

        _moveDirection = transform.position.x < _player.transform.position.x ? 1 : -1;
    }

    private void Update()
    {
        if (_hasToMove == true)
            transform.position += Vector3.right * _moveDirection * _moveSpeed * Time.deltaTime;
    }

    public void StopMovement() => _hasToMove = false;

    private void OnBecameInvisible()
    {
        Destroy(gameObject);
    }
}using System.Collections;
using System.Collections.Generic;
using UnityEngine;
public class PlatformMovement : MonoBehaviour
{
    private float _speed;

    public void SetSpeed(float speed)
    {
        _speed = speed;
    }

    private void Update()
    {
        
    }
}using System.Collections;
using System.Collections.Generic;
using UnityEngine;



public class PlatformSpawner : MonoBehaviour
{
    [SerializeField] private Transform[] _spawnPoints;
    [SerializeField] private GameObject _platformPrefab;
    [SerializeField] private float _verticalOffset = 0.5f;
    [SerializeField] private float _initialSpeed = 5f;
    [SerializeField] private float _speedMultiplier = 1.2f;

    private float? _lastPointPositionY = null;
    private float _currentSpeed;

    private void Start()
    {
        _currentSpeed = _initialSpeed;
        Spawn();
    }

    public void Spawn()
    {
        Transform randomSpawnPoint = _spawnPoints[Random.Range(0, _spawnPoints.Length)];
        float spawnPointPositionY = _lastPointPositionY == null ? randomSpawnPoint.position.y : (float)_lastPointPositionY + _verticalOffset;

        randomSpawnPoint.position = new Vector3(randomSpawnPoint.position.x, spawnPointPositionY, randomSpawnPoint.position.z);
        _lastPointPositionY = spawnPointPositionY;

        GameObject platform = Instantiate(_platformPrefab, randomSpawnPoint.position, Quaternion.identity);
       
        platform.GetComponent<PlatformMovement>().SetSpeed(_currentSpeed);

  
        _currentSpeed *= _speedMultiplier;
    }
}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

public class PlayerController : MonoBehaviour
{
    public UnityEvent Landed;
    public UnityEvent Dead;

    [SerializeField] private float _jumpForce;
    [SerializeField] private ContactFilter2D _platform;

    private Rigidbody2D _rigidbody;
    private bool _isOnPlatform => _rigidbody.IsTouching(_platform);

    private void Awake()
    {
        _rigidbody = GetComponent<Rigidbody2D>();
    }

    public void Jump()
    {
        if (_isOnPlatform == true)
            _rigidbody.AddForce(Vector2.up * _jumpForce, ForceMode2D.Impulse);
    }

    private void OnCollisionEnter2D(Collision2D collision)
    {
        GameObject collisionObject = collision.gameObject;

        if(collisionObject.transform.parent != null)
        {
            if (collisionObject.transform.parent.TryGetComponent(out Platform platform))
                platform.StopMovement();
        }

        if (collisionObject.CompareTag("PlatformWall"))
            Dead?.Invoke();
        else if(collisionObject.CompareTag("Platform"))
        {
            collisionObject.tag = "Untagged";
            Landed?.Invoke();
        }
    }
}using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class SceneLoader : MonoBehaviour
{
    public void ReloadScene()
    {
        string currentSceneName = SceneManager.GetActiveScene().name;
        SceneManager.LoadScene(currentSceneName);
    }
}
using System.Collections;
using System.Collections.Generic;
using TMPro;
using UnityEngine;

public class ScoreUI : MonoBehaviour
{
    private TextMeshProUGUI _field;
    private int _score = 0;

    private void Awake()
    {
        _field = GetComponent<TextMeshProUGUI>();
    }

    private void Start()
    {
        _field.text = _score.ToString();        
    }

    public void IncreaseScore()
    {
        _score += 1;
        _field.text = _score.ToString();
    }
}
