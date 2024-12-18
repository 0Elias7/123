import 'dart:async';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:image_picker/image_picker.dart';
import 'package:path_provider/path_provider.dart';

void main() {
  runApp(CameraApp());
}

class CameraApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Camera App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: CameraScreen(),
    );
  }
}

class CameraScreen extends StatefulWidget {
  @override
  _CameraScreenState createState() => _CameraScreenState();
}

class _CameraScreenState extends State<CameraScreen> {
  final ImagePicker _picker = ImagePicker();

  Future<void> _takePhoto() async {
    final XFile? photo = await _picker.pickImage(source: ImageSource.camera);

    if (photo != null) {
      _showSaveDialog(File(photo.path));
    }
  }

  Future<void> _showSaveDialog(File image) async {
    final TextEditingController folderController = TextEditingController();

    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Save Photo'),
        content: TextField(
          controller: folderController,
          decoration: InputDecoration(
            hintText: 'Enter folder name',
          ),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.of(context).pop(),
            child: Text('Cancel'),
          ),
          TextButton(
            onPressed: () async {
              String folderName = folderController.text.trim();
              if (folderName.isEmpty) {
                folderName = 'Default Folder';
              }

              await _saveImage(image, folderName);
              Navigator.of(context).pop();
            },
            child: Text('Save'),
          ),
        ],
      ),
    );
  }

  Future<void> _saveImage(File image, String folderName) async {
    final Directory appDir = await getApplicationDocumentsDirectory();
    final Directory folderDir = Directory('${appDir.path}/$folderName');

    if (!await folderDir.exists()) {
      await folderDir.create(recursive: true);
    }

    final String fileName = '${DateTime.now().millisecondsSinceEpoch}.jpg';
    final File newImage = await image.copy('${folderDir.path}/$fileName');

    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('Image saved to ${newImage.path}')),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Camera App'),
      ),
      body: Center(
        child: ElevatedButton(
          onPressed: _takePhoto,
          child: Text('Capture Photo'),
        ),
      ),
    );
  }
}
