import 'package:flutter/material.dart';
import 'package:hive_flutter/hive_flutter.dart';

void main() async {
  await Hive.initFlutter();
  await Hive.openBox('users');
  runApp(MaterialApp(home: MyApp()));
}

class MyApp extends StatelessWidget {
  final box = Hive.box('users');
  final email = TextEditingController();
  final senha = TextEditingController();

  void salvar(BuildContext ctx, [int? i]) {
    if (i != null) {
      final u = box.getAt(i);
      email.text = u['email'];
      senha.text = u['senha'];
    } else {
      email.clear();
      senha.clear();
    }

    showDialog(
      context: ctx,
      builder: (_) => AlertDialog(
        title: Text(i == null ? 'Novo' : 'Editar'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(controller: email, decoration: InputDecoration(labelText: 'Email')),
            TextField(controller: senha, decoration: InputDecoration(labelText: 'Senha')),
          ],
        ),
        actions: [
          TextButton(
            onPressed: () {
              final user = {'email': email.text, 'senha': senha.text};
              i == null ? box.add(user) : box.putAt(i, user);
              Navigator.pop(ctx);
            },
            child: Text('Salvar'),
          )
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Usuários')),
      body: ValueListenableBuilder(
        valueListenable: box.listenable(),
        builder: (_, Box box, __) => ListView.builder(
          itemCount: box.length,
          itemBuilder: (_, i) {
            final u = box.getAt(i);
            return ListTile(
              title: Text(u['email']),
              subtitle: Text(u['senha']),
              trailing: Row(
                mainAxisSize: MainAxisSize.min,
                children: [
                  IconButton(icon: Icon(Icons.edit), onPressed: () => salvar(context, i)),
                  IconButton(icon: Icon(Icons.delete), onPressed: () => box.deleteAt(i)),
                ],
              ),
            );
          },
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => salvar(context),
        child: Icon(Icons.add),
      ),
    );
  }
}
